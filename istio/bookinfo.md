<!-- toc -->
# Istio 的 Bookinfo Application 示例拆解

Istio 文档中给出了一个 [Bookinfo Application][1] 示例，这里拆解它的实现。

## Bookinfo APP 组成

Bookinfo APP 由四个子系统组成，分别是：

* productpage，产品页，展示图书信息，依赖 details 和 reviews
* details，提供图书详情
* reviews，提供用户评论
* ratings，图书的排行榜

这些子系统分别用 python、ruby、java、node 开发，其中 reviews 系统一共有三个版本：v1、v2、v3。

![Bookinfo Application](../img/envoy/bookinfo.svg)

四个服务部署在 Kubernetes 中，均是由 Service、ServiceAccount、Deployment 组成，其中 reviews 有三个 Deployment ：

### Detail service

```yaml
##################################################################################################
# Details service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: details
  labels:
    app: details
    service: details
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: details
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-details
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: details-v1
  labels:
    app: details
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: details
      version: v1
  template:
    metadata:
      labels:
        app: details
        version: v1
    spec:
      serviceAccountName: bookinfo-details
      containers:
      - name: details
        image: docker.io/istio/examples-bookinfo-details-v1:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
---
```

### Ratings service

```yaml
##################################################################################################
# Ratings service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: ratings
  labels:
    app: ratings
    service: ratings
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: ratings
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-ratings
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ratings-v1
  labels:
    app: ratings
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ratings
      version: v1
  template:
    metadata:
      labels:
        app: ratings
        version: v1
    spec:
      serviceAccountName: bookinfo-ratings
      containers:
      - name: ratings
        image: docker.io/istio/examples-bookinfo-ratings-v1:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
```

### Reviews service

```yaml
##################################################################################################
# Reviews service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: reviews
  labels:
    app: reviews
    service: reviews
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: reviews
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-reviews
```

Reviews 有三个版本的 Deployment，它们都隶属于同一个 Service，带有不同的 version 标签：

V1：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v1
  labels:
    app: reviews
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v1
  template:
    metadata:
      labels:
        app: reviews
        version: v1
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v1:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
```

V2:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v2
  labels:
    app: reviews
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v2
  template:
    metadata:
      labels:
        app: reviews
        version: v2
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v2:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
```

V3:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v3
  labels:
    app: reviews
    version: v3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v3
  template:
    metadata:
      labels:
        app: reviews
        version: v3
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v3:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
```

### Productpage services

```yaml
piVersion: v1
kind: Service
metadata:
  name: productpage
  labels:
    app: productpage
    service: productpage
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: productpage
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-productpage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage-v1
  labels:
    app: productpage
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: productpage
      version: v1
  template:
    metadata:
      labels:
        app: productpage
        version: v1
    spec:
      serviceAccountName: bookinfo-productpage
      containers:
      - name: productpage
        image: docker.io/istio/examples-bookinfo-productpage-v1:1.15.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
```

## 部署到 Istio 网格中

要把 Bookinfo Application 部署在 istio 网格中，仅仅在安装了 istio 的 kubernetes 创建应用是不行的。只有带有标签 `istio-injection=enabled` 的 namespace 中的服务，才会被 istio 纳入网格中。

所以，[文档][1] 中的第一步操作是打标签：

```sh
kubectl label namespace default istio-injection=enabled
```

然后在打了标签的 namespace 中创建应用：

```sh
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

如果不想为 namesapce 打标签，用下面的命令调整 yaml 文件：

```sh
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
```

## 创建 Gateway，边界 envoy 开始监听

创建 Gateway，指示边界 envoy 监听 80 端口，边界 envoy 的 80 端口将是 Bookinfo 的访问入口。

```sh
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

## 创建 VirtualService，转发请求

创建一个 VirtualService，将边界 envoy 收到的流量转发到 productpage ：

```sh
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

这样就可以通过边界 envoy 访问 bookinfo 服务了。

## 边界 envoy 的访问地址

边界 envoy 是 istio 的组件之一，是下面的两个服务：

```sh
$ kubectl -n istio-system get deployment | grep gateway
istio-egressgateway      1/1     1            1           45h
istio-ingressgateway     1/1     1            1           45h

$ kubectl -n istio-system get service | grep gateway
istio-egressgateway      ClusterIP      10.111.134.223   <none>     80/TCP,443/TCP,15443/TCP
istio-ingressgateway     LoadBalancer   10.101.187.91    <pending>  15020:31270/TCP,80:31380/TCP,
                                                                    443:31390/TCP,31400:31400/TCP,
                                                                    15029:31248/TCP,15030:30079/TCP,
                                                                    15031:30269/TCP,15032:30249/TCP,
                                                                    15443:31829/TCP
```

约定 istio-ingressgateway 处理进入网格的流量，istio-egressgateway 处理离开网格的流量（离开网格的流量的处理参考 [Engress Control](./egress.md)）。

从上面可以看到 istio-ingressgateway 使用的 LoadBalancer 模式，80 端口的访问地址是 31380。

如果是用 minikube 部署的 kubernetes，可以用下面的方式获取访问地址：

```sh
$ minikube service list
|---------------|------------------------|--------------------------------|
|   NAMESPACE   |          NAME          |              URL               |
|---------------|------------------------|--------------------------------|
| istio-system  | istio-ingressgateway   | http://192.168.99.100:31270    |
|               |                        | http://192.168.99.100:31380    |
|               |                        | http://192.168.99.100:31390    |
|               |                        | http://192.168.99.100:31400    |
|               |                        | http://192.168.99.100:31248    |
|               |                        | http://192.168.99.100:30079    |
|               |                        | http://192.168.99.100:30269    |
|               |                        | http://192.168.99.100:30249    |
|               |                        | http://192.168.99.100:31829    |
|---------------|------------------------|--------------------------------|
```

用下面的 url 访问 bookinfo，注意 Path 为 `productpage`，与 VirtualService 中的配置一致：

```sh
http://192.168.99.100:31380/productpage
```

![bookinfo网页](../img/envoy/bookinfo.png)

## 配置 DestinationRule

DestinationRule 不是必须的，它是对代理转发行为的更精细调控，是负载均衡策略，可以为每个服务创建一个对应的 DestinationRule。

```sh
$ kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

下面创建的 DestinationRule 将 Pod 按照 Label 进行了分组：

productpage：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
      version: v1
```

reviews:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
```

ratings：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ratings
spec:
  host: ratings
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v2-mysql
    labels:
      version: v2-mysql
  - name: v2-mysql-vm
    labels:
      version: v2-mysql-vm
```

details:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: details
spec:
  host: details
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

## 网格内的转发规则

到此，只配置了一个 VirtualService，定义了从边界 envoy 进来的流量的转发规则，那么网格内的服务（productpage、details、reviews、ratings）之间的请求如何转发？

前面为这四个服务分别创建了 DestinationRule，把它们的 pod 按照 label 分组了，但此刻这些分组没有任何效果。如果不停的刷新 bookinf 的页面，会发现页面在变化：

![bookinfo网页](../img/envoy/bookinfo.png)

![bookinfo网页](../img/envoy/bookinfo2.png)


网页的内容发生变化，是因为 reviews 服务有 v1、v2、v3 三个版本，访问 productpage 时，productpage 随机从 reviews 的三个版本的 Pod 中获取用户评论数据，所有会看到不同的页面。

这说明了两件事：

* 第一，网格内的所有服务默认都是通的，可以互相访问的，没有 VirtualService 也是通的；
* 第二，网格内的服务如果没有配置转发规则，那么就随机转发。

可以为网格内的每个服务创建一个 VirtualService，控制网格内服务间的转发规则，参考 [Apply a virtual service][2]：

```sh
$ kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

productpage 的 VirtualService：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - productpage
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
```

reviews 的 VirtualService：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
```

ratings 的 VirtualService：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
```

details 的 VirtualService：

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
        subset: v1
```

reviews 的 VirtualService 中明确指定了将请求转发到 `subset: v1` （reviews 的 DestinationRule 中配置的） ，这时候访问 bookinfo，页面不再变化。

![bookinfo网页](../img/envoy/bookinfo3.png)

## 更多操作

[Traffic Management][3] 中有更多示范操作，譬如按照 user 转发、错误注入、流量整形、断路器、流量复制等，这些内容以后单独整理。

## 参考

[1]: https://istio.io/docs/examples/bookinfo/ "Bookinfo Application"
[2]: https://istio.io/docs/tasks/traffic-management/request-routing/ "Apply a virtual service"
[3]:  https://istio.io/docs/tasks/traffic-management/request-routing/ "Traffic Management"

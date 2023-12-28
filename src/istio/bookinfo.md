<!-- toc -->
# istio 的 Bookinfo Application 示例拆解

Istio 文档中给出了一个 [Bookinfo Application][1] 示例，这里拆解它的实现。

## Bookinfo APP 组成

Bookinfo APP 由四个子系统组成，分别是：

* productpage，产品页，展示图书信息，依赖 details 和 reviews
* details，图书详情查询
* reviews，用户评论查询，依赖 ratings
* ratings，图书排行榜查询

这些子系统分别用 python、ruby、java、node 开发，其中 reviews 系统有三个版本：v1、v2、v3。

![Bookinfo Application](../img/envoy/bookinfo.svg)

示例中把上述四个系统部署在 Kubernetes 中，每个系统由 Service、ServiceAccount、Deployment 组成。reviews 有三个版本，对应三个 Deployment。

## 部署到 istio 网格

要把 Bookinfo Application 部署在 istio 网格中，仅仅在安装了 istio 的 kubernetes 创建应用是不行的，需要在带有 `istio-injection=enabled` 标签的 namespace 中创建才可以。

所以 [文档][1] 中的第一步操作在目标 namespace 上打标签：

```sh
kubectl label namespace default istio-injection=enabled
```

然后在打了标签的 namespace 中创建应用：

```sh
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

如果不想给 namesapce 打标签，可以用下面的命令调整 yaml 文件：

```sh
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
```

### 验证 Productpage

部署完成后，验证 productpage，在集群内用它的 cluster ip 或者 pod ip 访问：

```sh
$ kubectl get svc productpage -o wide
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE   SELECTOR
productpage   ClusterIP   10.110.29.37     <none>        9080/TCP   76d   app=productpage
```

```sh
$ curl 10.110.29.37:9080
...省略...
  </body>
</html>
```

或者在 Productpage 等容器内安装 curl，用 curl 验证：

```sh
$ kubectl exec -it productpage-v1-667bc85676-sgbqp /bin/sh
$ apt-get update
$ apt-get install -y curl
$ curl http://productpage:9080
...省略...
  </body>
</html>
```

Productpage 是 bookinfo app 的入口，使用 python 开发，从代码中找到另外几个服务的接口：

```sh
$ kubectl exec -it productpage-v1-8554d58bff-wlkg7 cat  productpage.py >/tmp/productpage.py
```

查看代码得知其它几个服务的 http 接口，后面将使用这些接口验证服务：

```sh
details:  http://details:9080/details/0
ratings:  http://ratings:9080/ratings/0
reviews:  http://reviews:9080/reviews/0
```

Productpage service 部署 yaml：

```yaml
apiVersion: v1
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

### 验证 Detail

完成部署后，验证 details 服务：

```sh
$ kubectl get svc details -o wide
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE   SELECTOR
details   ClusterIP   10.107.162.106   <none>        9080/TCP   76d   app=details
```

```sh
$ curl http://10.107.162.106:9080/details/0
{"id":0,"author":"William Shakespeare","year":1595,"type":"paperback","pages":200,"publisher...省略...
```

details 服务的部署文件：

```yaml
##################################################################################
# Details service
###################################################################################
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

### 验证 Ratings

完成部署后，验证 ratings 服务：

```sh
$ kubectl get svc ratings -o wide
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE   SELECTOR
ratings   ClusterIP   10.107.67.216   <none>        9080/TCP   76d   app=ratings
```

```sh
$ curl http://10.107.67.216:9080/ratings/0
{"id":0,"ratings":{"Reviewer1":5,"Reviewer2":4}}$
```

ratings 服务的部署文件：

```yaml
###################################################################################
# Ratings service
##################################################################################
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

### 验证 Reviews

完成部署后，验证 reviews 服务：

```sh
$ kubectl get svc reviews -o wide
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE   SELECTOR
reviews   ClusterIP   10.100.249.106   <none>        9080/TCP   76d   app=reviews
```

```sh
$ curl http://10.100.249.106:9080/reviews/0
{"id": "0","reviews": [{  "reviewer": "Reviewer1",  "text": "An extremely entertaining play by Shakespeare....省略...
```

Reviews 有三个版本，对应三个 Deployment，隶属于同一个 Service：

```yaml
###################################################################################
# Reviews service
###################################################################################
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

三个版本的 deployment：

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

## 创建 Gateway，ingress envoy 开始监听

创建 Gateway，指示边界 envoy 监听 80 端口，允许通过 ingress envoy 的 80 端口访问 Bookinfo：

```sh
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

Gateway 定义：

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

如果这时候还没有创建 VirtualService，ingress envoy 还不知道要转发到哪里。

## 创建 VirtualService，转发请求

创建 VirtualService，指示 ingress envoy 将外部到来的请求转发到 productpage ：

```sh
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml  
```

VirtualService 定义：

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

这时可以通过边界 envoy 访问 bookinfo。这里的 hosts 配置是 `*`，可以根据需要改成具体的域名。

## 通过边界 envoy 访问应用

边界 envoy 是 istio 的组件之一，由两个服务组成：

```sh
$ kubectl -n istio-system get deployment | grep gateway
istio-egressgateway      1/1     1            1           45h
istio-ingressgateway     1/1     1            1           45h
```

一个负责从外部流入网格的流量（ingress），一个负责从网格流向外部的流量（egress）。

约定 istio-ingressgateway 处理外部进入网格的流量，istio-egressgateway 处理网格流向外部的流量，离开网格的流量的处理参考 [Engress Control](./egress.md)。

通过 kubernetes 提供的方式访问 istio-ingressgateway：

```sh
$ kubectl -n istio-system get service | grep gateway
istio-egressgateway      ClusterIP      10.111.134.223   <none>     80/TCP,443/TCP,15443/TCP
istio-ingressgateway     LoadBalancer   10.101.187.91    <pending>  15020:31270/TCP,80:31380/TCP,
                                                                    443:31390/TCP,31400:31400/TCP,
                                                                    15029:31248/TCP,15030:30079/TCP,
                                                                    15031:30269/TCP,15032:30249/TCP,
                                                                    15443:31829/TCP
```

可以看到 istio-ingressgateway 使用 LoadBalancer 模式，它的 80 端口对应的映射端口是 31380。

如果是用 minikube 部署的 kubernetes，可以用下面的方式获取 istio-ingressgateway 的访问地址：

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

这里使用了 minikube，用下面的 url 访问 bookinfo，path 为 `productpage`，与 VirtualService 中的配置一致：

```sh
http://192.168.99.100:31380/productpage
```

![bookinfo网页](../img/envoy/bookinfo.png)

虽然可以通过边界 envoy 访问应用，但是因为没有设置转发策略，四个子系统之间的互相访问和通过 kubernetes 中的 svc 访问效果相同。

刷新页面会看到页面在变化：

![bookinfo网页](../img/envoy/bookinfo.png)

![bookinfo网页](../img/envoy/bookinfo2.png)

这是因为 reviews 服务有 v1、v2、v3 三个版本，访问 productpage 时，productpage 随机从 reviews 的三个版本的 pod 中获取用户评论数据，所以会看到不同的页面。

可以为每个服务创建一个 VirtualService 和 Destination，精细管控每个服务的转发策略。

## 设置网格内的转发策略：DestinationRule

为每个服务创建一个 DestinationRule，设置转发策略：

```sh
$ kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

这里在 DestinationRule 中将 Pod 按照 Label 进行了分组，拆分成多个 subnet：

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

转发策略被 VirtualService 引用后才生效，参考 [Apply a virtual service][2]，还需要为每个服务创建一个 VirtualService，在 VirtualService 中引用 DestinationRule 中的 subset：

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

details 的 VirtualService：

```yaml
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

reviews 的 VirtualService 绑定了 reviews 的 `subset: v1`，这时候访问 bookinfo，页面不再变化。

![bookinfo网页](../img/envoy/bookinfo3.png)

## 按照用户转发

更新 reviews 的 VirtualService，将 jason 用户的请求转发到 v2 版本：

```sh
$ kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

点击界面右上角的 login，以用户 jason 的身份登录（密码为空），然后刷新页面，会发现页面变成了 v2 版本：

![bookinfo网页](../img/envoy/bookinfo4.png)

退出 jason 账号后，页面又回到了 v1 版本。

## 更多操作

[Traffic Management][3] 中有更多示范操作，譬如按照 user 转发、错误注入、流量整形、断路器、流量复制等，这些内容放在后面单独整理。

## 参考

[1]: https://istio.io/docs/examples/bookinfo/ "Bookinfo Application"
[2]: https://istio.io/docs/tasks/traffic-management/request-routing/ "Apply a virtual service"
[3]:  https://istio.io/docs/tasks/traffic-management/request-routing/ "Traffic Management"

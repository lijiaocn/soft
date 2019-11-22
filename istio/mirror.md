<!-- toc -->
# istio 的流量复制功能

istio 的流量复制在 [VirtualService](./vsvc.md) 中设置，istio 1.4.x 开始支持。

## 创建正常接收请求的服务

创建 http-record-v1 接收请求：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: http-record
  labels:
    app: http-record
spec:
  ports:
  - name: http
    port: 8000
    targetPort: 8080
  selector:
    app: http-record
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: http-record-v1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: http-record
        version: v1
    spec:
      containers:
      - image: lijiaocn/http-record:0.0.1
        imagePullPolicy: IfNotPresent
        name: http-record
        ports:
        - containerPort: 8080
```

创建对应的 VirtualService 和 DestinationRule：

```yaml
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: http-record
spec:
  hosts:
    - http-record
  http:
  - route:
    - destination:
        host: http-record
        subset: v1
      weight: 100
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: http-record
spec:
  host: http-record
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

进入一个 Pod 中，验证 http-record 服务：

```sh
$ curl http-record:8000
{
    "RemoteAddr": "127.0.0.1:50014",
    "Method": "GET",
    "Host": "http-record:8000",
    "RequestURI": "/",
    "Header": {
        "Accept": [
...省略...
```

## 创建接收复制流量的版本

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: http-record-v2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: http-record
        version: v2
    spec:
      containers:
      - image: lijiaocn/http-record:0.0.1
        imagePullPolicy: IfNotPresent
        name: http-record
        ports:
        - containerPort: 8080
```

## 设置镜像复制策略

编辑名为 http-record 的 VirtualService，设置流量复制策略，将 100% 的流量复制到 v2： 

```yaml
$ kubectl edit vs http-record
  hosts:
  - http-record
  http:
  - mirror:
      host: http-record
      subset: v2
    mirror_percent: 100
    route:
    - destination:
        host: http-record
        subset: v1
      weight: 100
```

查看 http-record-v2 的日志，会发现 http-record 时，http-record-v2 的 Pod 收到了同样的请求：


## 镜像到另一个服务

```
$ kubectl edit vs productpage
...省略...
  hosts:
  - productpage
  http:
  - mirror:
      host: http-record
      subset: v2
    mirror_percent: 100
    route:
    - destination:
        host: productpage
        subset: v1
...省略...
```

然后查看 http-record 日志，会发现访问 productpage 时，http-record 收到了一份相同的请求：

```sh
$ kubectl logs -f http-record-685f56d9f4-6k64z http-record
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://istio.io/news/releases/1.4.x/announcing-1.4/change-notes/ "1.4.x Change Notes"

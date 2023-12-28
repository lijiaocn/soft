<!-- toc -->
# 在 istio 中部署回显 http 请求的 http-record 服务

这是本手册设计的一个服务示例，http-record 的功能特别简单，将收到的请求原封不断回显，同时在标准输出中打印。它和 echoserver 服务的功能类似，但是多了在标准输出中打印的功能，在观察流量复制效果时特别有用。

http-record 容器的用法见 [常用工具-HTTP协议相关](../tools/http.md)。

## 为目标 namespace 打上 label

仅仅在安装了 istio 的 kubernetes 创建应用是不行的，需要在带有 `istio-injection=enabled` 标签的 namespace 中创建才可以。

这里要在 default 中部署 http-record ，为 default namespace 上打标签：

```sh
kubectl label namespace default istio-injection=enabled
```

[Bookinfo Application](./bookinfo.md) 对此步操作有更详细的解释。

## 部署 http-record 

用下面的 yaml 创建 http-record 的 Deployment 和 Service：

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

## 配置 http-record 的 istio 策略

用下面的 yaml 创建 http-record 的 istio 策略：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: http-record
spec:
  gateways:
  - http-record-gateway
  hosts:
    - http-record.example
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
---
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: http-record-gateway
  namespace: default
spec:
  selector:
    istio: ingressgateway
  servers:
  - hosts:
    - 'http-record.example'
    port:
      name: http
      number: 80
      protocol: HTTP
```

## 效果

通过 istio 的 ingressgateway 访问 http-record 服务，域名为 http-record.example：

```sh
$ curl  -H "Host: http-record.example" 192.168.99.100:31380
{
    "RemoteAddr": "127.0.0.1:52454",
    "Method": "GET",
    "Host": "http-record.example",
    "RequestURI": "/",
    "Header": {
        "Accept": [
            "*/*"
        ],
        "Content-Length": [
            "0"
        ],
        "User-Agent": [
            "curl/7.54.0"
        ],
        "X-B3-Parentspanid": [
            "afc976737b24ece3"
        ],
        "X-B3-Sampled": [
            "1"
        ],
...省略...
```


## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

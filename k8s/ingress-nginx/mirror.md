<!-- toc -->
# Ingress-nginx 的请求复制功能

ingress-nginx 支持请求复制功能，将同一域名下指定 path 上的请求复制到另一个 path。

```sh
cd 07-mirror
```

## 部署接收复制请求的服务

创建一个名为 http-record 的服务，用来接收复制的请求：

```sh
$ kubectl -n demo-echo apply -f http-record.yaml
namespace/demo-echo unchanged
service/http-record created
deployment.apps/http-record created
```

```sh
$ kubectl -n demo-echo get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                     AGE
echo          NodePort    10.111.29.87    <none>        80:30411/TCP,22:31867/TCP   47d
http-record   NodePort    10.106.66.216   <none>        80:31734/TCP,22:32324/TCP   29s
```

## 配置被复制的服务的 ingress

需要创建两个 ingress，两个 ingress 使用相同的 host。

第一个 ingress 用来接收复制的请求，指向 http-record 服务：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-mirror-backend
spec:
  rules:
  - host: mirror.echo.example
    http:
      paths:
      - path: /echo
        backend:
          serviceName: http-record
          servicePort: 80
```

第二个 ingress 配置要被复制的请求，指向目标服务。mirror-uri 接收复制的请求的地址，对 ingress 中所有 path 有效：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-mirror
  annotations:
    nginx.ingress.kubernetes.io/mirror-uri: "/echo"
spec:
  rules:
  - host: mirror.echo.example
    http:
      paths:
      - path: /
        backend:
          serviceName: echo
          servicePort: 80
```

默认将请求的 body 一同复制，如果不想复制 body，在第二个 ingress 中用下面的注解关闭：

```sh
nginx.ingress.kubernetes.io/mirror-request-body: "off"
```

## 请求复制效果

发起请求：

```sh
$ curl -X POST -d "111111" -H "Host: mirror.echo.example" "192.168.99.100:30933/aaaaaa/bbbb?c=a"
Hostname: echo-597d89dcd9-m84tq

Pod Information:
	-no pod information available-
...
```

上面的请求将被复制一份到 http-record 容器，http-record 容器的回应被 ingress 忽略，在 http-record 的日志中可以看到复制来的请求：

```sh
$ kubectl -n demo-echo logs -f http-record-66478bdbb7-xslzg
```

http-record 容器收到的请求信息，注意原始的 uri 使用 header 传递的 —— **X-Original-Uri**：

```json
{
    "RemoteAddr": "172.17.0.11:59784",
    "Method": "POST",
    "Host": "mirror.echo.example",
    "RequestURI": "/echo?c=a",
    "Header": {
        "Accept": [
            "*/*"
        ],
        "Content-Length": [
            "6"
        ],
        "Content-Type": [
            "application/x-www-form-urlencoded"
        ],
        "User-Agent": [
            "curl/7.54.0"
        ],
        "X-Forwarded-For": [
            "172.17.0.1"
        ],
        "X-Forwarded-Host": [
            "mirror.echo.example"
        ],
        "X-Forwarded-Port": [
            "80"
        ],
        "X-Forwarded-Proto": [
            "http"
        ],
        "X-Original-Uri": [
            "/aaaaaa/bbbb?c=a"
        ],
        "X-Real-Ip": [
            "172.17.0.1"
        ],
        "X-Request-Id": [
            "86e25adfcd7f2a673925d9a17769272a"
        ],
        "X-Scheme": [
            "http"
        ]
    },
    "Body": "1111"
```

## 参考

1. [李佶澳的博客][1]
2. [mirror][2]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#mirror "mirror"

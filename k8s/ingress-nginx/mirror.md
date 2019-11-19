<!-- toc -->
# ingress-nginx 的请求复制功能

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

## 复制原始的 uri

上面的复制效果，复制后的请求的 uri 为 /echo，不是原始的 uri。在下面的文章中探讨了这个问题：

* [kubernetes ingress-nginx http 请求复制功能与 nginx mirror 的行为差异][3]。

现在发现了一个更好的解决方法，直接在接收复制流量的 ingress 中加一个 rewrite 就可以了，如下：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    # 使用原始的 uri
    nginx.ingress.kubernetes.io/rewrite-target: $request_uri
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

这时候访问：

```sh
$ curl -H "Host: mirror.echo.example" "192.168.99.100:30933/dddd?a=1&b=2"
```

复制端看到的请求的 uri 是 dddd?a=1&b=2 ：

```json
{
    "RemoteAddr": "172.17.0.27:39254",
    "Method": "GET",
    "Host": "mirror.echo.example",
    "RequestURI": "/dddd%3Fa=1\u0026b=2?a=1\u0026b=2",
    ...省略...
}
```

## 补充：另一种实现方法，可以按比例复制

阿里云提供的一种实现方法，详情见 [通过K8S Ingress Controller来实现应用的流量复制][4]。

在 ingress-nginx 使用的 configmap nginx-configuration 中添加配置，通过 [http-snippet][7] 添加全局配置，设置复制的流量的去处。流量的去处可以是 IP 地址或者域名，下面将使用 http-record 服务的 cluster ip：

```sh
$ kubectl -n demo-echo get svc
NAME          TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                     AGE
echo          NodePort   10.100.105.128   <none>        80:32110/TCP,22:32561/TCP   33d
http-record   NodePort   10.106.66.216    <none>        80:31734/TCP,22:32324/TCP   29d
```

在 configmap nginx-configuration 中添加的配置，这个配置是全局的，会被注入到 nginx.conf：

```yaml
 http-snippet: |
   split_clients "$date_gmt" $mirror_echo_to_http_record {
      100%    10.106.66.216; # 可以是域名、IP，100% 是复制比例
   }
```

split_clients 指令的用途见 [nginx 的 A/B 测试功能](../../nginx/abtest.md)，这个指令的强大之处是可以按 hash 值的分布区间设置变量值。hash 算法的输入字符串自行指定，这里是 $date_gmt，使用源IP、request_uri 等 nginx 支持的变量都是可以的。

在需要被复制的 ingress 中用 [configuration-snippet][5] 和 [server-snippet][6] 注入配置，这些配置会 **原封不动** 的合并到最终的 nginx.conf 中， **不要有错误**：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-mirror2
  annotations:
    nginx.ingress.kubernetes.io/configuration-snippet: |
        mirror /mirror334556; # 配置多次该项则可放大N倍流量
    nginx.ingress.kubernetes.io/server-snippet: |
        location = /mirror334556 {
            internal;
            access_log off; # 关闭mirror请求日志
            set $proxy_host $host;
            proxy_pass http://$mirror_echo_to_http_record$request_uri;
        }
spec:
  rules:
  - host: mirror2.echo.example
    http:
      paths:
      - path: /
        backend:
          serviceName: echo
          servicePort: 80
```

注意 $mirror_echo_to_http_record 要和 configmap nginx-configuration 中配置的配对，mirror 指定的 /mirror334556 和同名的 location 对应。

访问 mirror2.echo.example，会看到请求被复制到 http-record：

```sh
$ curl -H "Host: mirror2.echo.example" 192.168.99.100:30933/1234

Hostname: echo-597d89dcd9-m84tq

Pod Information:
	-no pod information available-
```

http-record 收到的复制请求：

```sh
/go/src/Server/echo.go:46: {
    "RemoteAddr": "10.0.2.15:39222",
    "Method": "GET",
    "Host": "mirror2.echo.example",
    "RequestURI": "/1234",
    "Header": {
        "Accept": [
            "*/*"
        ],
        "Connection": [
            "close"
        ],
        "User-Agent": [
            "curl/7.54.0"
        ]
    },
    "Body": ""
}
```

这种方法相比第一种方式，功能更强大，是一种更巧妙更简洁的做法，工作原理：

1. [结合 split_clients 实现请求的部分复制](../../nginx/mirror.md)。
2. [Nginx 的 A/B 测试功能](../../nginx/abtest.md)

## 参考

1. [李佶澳的博客][1]
2. [mirror][2]
3. [kubernetes ingress-nginx http 请求复制功能与 nginx mirror 的行为差异][3]
4. [通过K8S Ingress Controller来实现应用的流量复制][4]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#mirror "mirror"
[3]: https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2019/10/21/ingress-nginx-request-mirror.html
[4]: https://yq.aliyun.com/articles/665338 "通过K8S Ingress Controller来实现应用的流量复制"
[5]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#configuration-snippet "configuration-snippet"
[6]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#server-snippet   "server-snippet"
[7]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#http-snippet "http-snippet"

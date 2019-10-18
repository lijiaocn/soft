<!-- toc -->
# ingress-nginx 针对源 IP 限速

ingress-nginx 可以限制源 IP 的最大连接数、每秒请求数、每分钟请求数、每秒发送的数据量、限速白名单。超过上限的客户端会收到 ingress-nginx 返回的 503。

```sh
cd 08-ratelimit
```

## 配置 ingress

优先级： limit-connections >  limit-rpm > limit-rps。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-ratelimit
  annotations:
    # 每个源 IP 可以建立最大连接数
    nginx.ingress.kubernetes.io/limit-connections: 2
    # 每个源 IP 每分钟最大请求次数
    nginx.ingress.kubernetes.io/limit-rpm: "5"
    # 每个源 IP 每秒最大请求次数
    nginx.ingress.kubernetes.io/limit-rps: "1"
spec:
  rules:
  - host: ratelimit.echo.example
    http:
      paths:
      - path: /
        backend:
          serviceName: echo
          servicePort: 80
```

执行：

```sh
$ kubectl -n demo-echo apply -f ratelimit-echo-example-ingress.yaml
```

## 限速效果

连续访问超速后，ingress-nginx 返回 503：

```sh
$ curl -v  -H "Host: ratelimit.echo.example" "192.168.99.100:30933/"
*   Trying 192.168.99.100...
* TCP_NODELAY set
* Connected to 192.168.99.100 (192.168.99.100) port 30933 (#0)
> GET / HTTP/1.1
> Host: ratelimit.echo.example
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 503 Service Temporarily Unavailable
< Server: openresty/1.15.8.1
< Date: Wed, 16 Oct 2019 10:37:41 GMT
< Content-Type: text/html
< Content-Length: 203
< Connection: keep-alive
```

## 参考

1. [李佶澳的博客][1]
2. [rate-limiting][2]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#rate-limiting "rate-limiting"

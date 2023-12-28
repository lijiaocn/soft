<!-- toc -->
# Ingress-nginx 的 uri 改写功能（rewrite）

ingress-nginx 支持 uri 改写，并且支持正则捕获。

```sh
cd 06-rewrite
```

## 配置目标应用的 ingress

创建一个 ingress，path 匹配规则为 `/rewrite/(.*)`，rewrite-target 中可以使用 path 中的正则匹配：

```sh
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-rewrite
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - host: rewrite.echo.example
    http:
      paths:
      - path: /rewrite/(.*)
        backend:
          serviceName: echo
          servicePort: 80
```

需要注意 rewrite-target 对 ingress 中的所有 path 有效。

创建：

```sh
$ kubectl -n demo-echo create -f rewrite-echo-example-ingress.yaml
```

## 效果

访问 `/rewrite/abc`，服务端看到的是改写后的 `/abc`：

```sh
$ curl  -H "Host: rewrite.echo.example" 192.168.99.100:30933/rewrite/abc

Hostname: echo-597d89dcd9-4dp6f

Pod Information:
    -no pod information available-

Server values:
    server_version=nginx: 1.13.3 - lua: 10008

Request Information:
    client_address=172.17.0.11
    method=GET
    real path=/abc    #<--改写后的 path
    query=
```

## 参考

1. [李佶澳的博客][1]
2. [Rewrite][2]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.github.io/ingress-nginx/examples/rewrite/ "Rewrite"

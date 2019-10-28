<!-- toc -->
# ingress-nginx 的金丝雀（canary）发布功能

之前有过一篇笔记（[kubernetes ingress-nginx 的金丝雀（canary）/灰度发布功能的使用方法][2]），这里重新整理下。什么是金丝雀、什么是蓝绿、什么是灰度，见 [蓝绿部署、金丝雀发布（灰度发布）、A/B测试的准确定义][3]。

## 原有的 ingress 不需要改变

使用 canary 功能时，主版本 ingress 保持原状，不需要任何改动。

这里使用下面 ingress 作为主版本 ingress： 

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-canary-master
spec:
  rules:
  - host: canary.echo.example
    http:
      paths:
      - path: /
        backend:
          serviceName: echo
          servicePort: 80
```

执行：

```sh
kubectl -n demo-echo create -f canary-master-echo-example-ingress.yaml
```

## 创建 canary （金丝雀）ingress

金丝雀 ingress 和主 ingress 使用相同的 host 和 path，区别在于多出了一些 annotation：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-canary-version
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "version"
    nginx.ingress.kubernetes.io/canary-by-header-value: "canary"
    nginx.ingress.kubernetes.io/canary-by-cookie: "canary-cookie"
    nginx.ingress.kubernetes.io/canary-weight: "50"
spec:
  rules:
  - host: canary.echo.example
    http:
      paths:
      - path: /
        backend:
          serviceName: http-record
          servicePort: 80
```

执行：

```sh
kubectl -n demo-echo create -f canary-version-echo-example-ingress.yaml
```

`nginx.ingress.kubernetes.io/canary: true` 是启用 canary，其余几个 annotation 分别是设置导向 cannary 的条件、权重。

## 金丝雀效果

使用下面的方式时，50% 的请求转发到金丝雀服务（对应设置 canary-weight）：

```sh
$ curl -H "Host: canary.echo.example"  192.168.99.100:30933/
```

带有 "version: canary" 头的请求，转发到金丝雀服务（对应设置 canary-by-header 和 canary-by-header-value）：

```sh
$ curl -H "Host: canary.echo.example" -H "version: canary" 192.168.99.100:30933/
```

带有 canary-cookie 且 cookie  值为 always 的请求，转发到金丝雀服务（对应设置 canary-by-cookie）：

```sh
$ curl -H "Host: canary.echo.example" -b canary-cookie=always 192.168.99.100:30933/
```

带有 canary-cookie 且 cookie  值为 always 的请求，转发到主服务（对应设置 canary-by-cookie）：

```sh
$ curl -H "Host: canary.echo.example" -b canary-cookie=never 192.168.99.100:30933/
```

header、cookie、weight 的优先级：canary-by-header -> canary-by-cookie -> canary-weight。

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2019/07/12/ingress-nginx-canary.html "kubernetes ingress-nginx 的金丝雀（canary）/灰度发布功能的使用方法"
[3]: https://www.lijiaocn.com/%E6%96%B9%E6%B3%95/2018/10/23/devops-blue-green-deployment-ab-test-canary.html "蓝绿部署、金丝雀发布（灰度发布）、A/B测试的准确定义"

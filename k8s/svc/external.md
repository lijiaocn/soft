# Kubernetes 导入外部服务 

有一个位于集群外部的服务，地址为 192.168.99.1:4180，将这个服务导入到 kubernetes 中。

1 为外部服务创建一个对应的 Service：

```yaml
kind: Service
apiVersion: v1
metadata:
 name: external-github-oauth-proxy
spec:
 type: ClusterIP
 ports:
 - port: 4180
   targetPort: 4180
```

2 创建一个与 Service 同名的 endpoint，填入外部服务的 IP：

```yaml
kind: Endpoints
apiVersion: v1
metadata:
 name: external-github-oauth-proxy
subsets:
 - addresses:
     - ip: 192.168.99.1
   ports:
     - port: 4180
```

然后可以像使用其它服务一样使用外部服务，譬如在 ingress 中配置外部服务：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-auth-oauth2-ext-proxy
spec:
  rules:
  - host: auth-oauth2-ext.echo.example
    http:
      paths:
      - path: /oauth2
        backend:
          serviceName: external-github-oauth-proxy
          servicePort: 4180
```

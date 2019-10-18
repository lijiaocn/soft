<!-- toc -->
# Kubernetes 导入外部服务 

本文使用的 yaml 文件：

```sh 
git clone https://github.com/introclass/kubernetes-yamls
cd kubernetes-yamls/ingress-nginx/05-1-external-svc
```

## 启动一个外部服务

启动一个位于集群外部的服务：

```sh
./start-github-oauth2-proxy.sh
```

这里的服务地址为 192.168.99.1:4180，将这个服务导入到 kubernetes 中。

## 导入到 kubernetes 中

为外部服务创建 Service 和一个与 Service 同名的 endpoint，endpoint 中填入外部服务的 IP。

名为 external-github-oauth-proxy 的 Service：

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

同名的 endpoint：

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

创建：

```sh
$ kubectl -n demo-echo create -f external-github-oauth2-proxy.yaml
```

## 导入效果

导入后可以像使用其它服务一样使用外部服务，譬如在 ingress 中配置外部服务：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: github-oauth2-proxy
spec:
  rules:
  - host: github-oauth2.example
    http:
      paths:
      - backend:
          serviceName: github-oauth2-proxy
          servicePort: 4180
        path: /
```

创建：

```sh
$ kubectl -n demo-echo create github-oauth-proxy.yaml
```

## 参考 

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

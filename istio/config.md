<!-- toc -->
# istio 的配置参数

istio 的配置参数非常非常多！

用 [helm](./demo-install.md) 部署时，通过 --set 设置，如下：

```sh
$ kubectl create namespace istio-system
$ helm template install/kubernetes/helm/istio-init \
  --name istio-init \
  --namespace istio-system \
  --set gateways.istio-ingressgateway.type=NodePort \
  | kubectl apply -f -
```

如果用 istioctl 部署，用下面的命令生成部署文件，查看配置参数：

```sh
./istioctl manifest generate
./istioctl profile dump
```

## 参考

1. [李佶澳的博客][1]
[1]: https://www.lijiaocn.com "李佶澳的博客"

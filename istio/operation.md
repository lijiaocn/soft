# Istio 的操作

[istioctl][1]  是 istio 的管理命令，用来调试 istio，或诊断存在的问题。istioctl 管理的是部署在 kubernetes 中的 istio，它通过读取本地 kubeconfig context，获取的 kubernetes 的地址以及操作权限。

istioctl 命令位于 istio-1.2.5/bin 中，有多个子命令：

```sh
--context <string>                 The name of the kubeconfig context to use (default ``)
--istioNamespace <string>    -i    Istio system namespace (default `istio-system`)
...
```

## 参考

[1]: https://istio.io/docs/reference/commands/istioctl/  "istioctl"

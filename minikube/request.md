<!-- toc -->
# 访问 minikube 启动的 kubernetes 中的服务

Kubernetes 在本地虚拟机中运行，如果要访问部署其中的服务，需要通过 node port。

## 通过 node port 访问

在 kubernetes 中设置了使用 node port 的 service 后，命令 minikube service list 会列出这些服务的本地访问地址：

```
$ minkube service list
|---------------|-------------------|--------------------------------|
|   NAMESPACE   |       NAME        |              URL               |
|---------------|-------------------|--------------------------------|
| default       | kubernetes        | No node port                   |
| demo-echo     | echo              | http://192.168.99.100:30801    |
|               |                   | http://192.168.99.100:30263    |
| demo-webshell | webshell          | No node port                   |
| demo-webshell | webshell-nodeport | http://192.168.99.100:32635    |
|               |                   | http://192.168.99.100:32454    |
| kube-system   | kube-dns          | No node port                   |
|---------------|-------------------|--------------------------------|
```

## 参考

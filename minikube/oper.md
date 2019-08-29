<!-- toc -->
# 操作用 minikube 启动的 kubernetes

Minikube 在启动 kubernetes 时会设置  ~/.kube/config 文件，在其中增加了访问凭证：

```yaml
...省略...
users:
- name: minikube
  user:
    client-certificate: /Users/lijiao/.minikube/client.crt
    client-key: /Users/lijiao/.minikube/client.key
...省略...
```

所以在当前机器上可以用 kubectl 直接操作 kubernetes：

```sh
$ kubectl get ns
NAME              STATUS   AGE
kube-node-lease   Active   7d19h
kube-public       Active   7d19h
kube-system       Active   7d19h
```

## 参考

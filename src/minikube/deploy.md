<!-- toc -->
# 用 minikube 启动 kubernetes 

Minikube 可以部署多个版本的 kubernetes，如果不指定版本默认使用最新版。

## 启动默认版本的 kubernetes

```sh
$ minkube start
😄  minikube v1.3.1 on Darwin 10.14
🔥  Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.15.2 on Docker 18.09.8 ...
    ▪ scheduler.leader-elect-resource-lock=configmaps
    ▪ controller-manager.leader-elect-resource-lock=configmaps
🚜  Pulling images ...
🚀  Launching Kubernetes ...
⌛  Waiting for: apiserver proxy

💣  Wait failed: waiting for k8s-app=kube-proxy: timed out waiting for the condition

😿  Sorry that minikube crashed. If this was unexpected, we would love to hear from you:
👉  https://github.com/kubernetes/minikube/issues/new/choose
```

如果遇到 "Wait failed" 的情况先别急着重试，即使遇到了上面的错误，kubernetes 也可能已经启动成功了：

```sh
$ kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

## 启动指定版本的 kubernetes

--kubernetes-version 指定要部署的 kubernetes 版本：

```sh
$ minikube start --kubernetes-version v1.12.0
😄  minikube v1.3.1 on Darwin 10.14
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🔄  Starting existing virtualbox VM for "minikube" ...
```

虚拟化软件默认为 virtualbox，如果要用其他的虚拟机化，用 --vm-driver 指定。

支持的参数在 minkube help start 中可以看到：

```sh
$ ./minkube help start
Starts a local kubernetes cluster

Options:
      --apiserver-ips=[]: A set of apiserver IP Addresses which are used in the generated certificate for kubernetes.
This can be used if you want to make the apiserver available from outside the machine
      --apiserver-name='minikubeCA': The apiserver name which is used in the generated certificate for kubernetes.  This
	  ...省略...
```

## 关闭 kubernetes

```sh
$ minikube stop
✋  Stopping "minikube" in virtualbox ...
🛑  "minikube" stopped.
```

## 删除

```sh
$ minikube delete
```

## 参考


[1]: https://godoc.org/k8s.io/kubernetes/pkg/scheduler/apis/config#KubeSchedulerConfiguration "KubeSchedulerConfiguration"
[2]: https://godoc.org/k8s.io/kubernetes/pkg/kubelet/apis/config#KubeletConfiguration "KubeletConfiguration"
[3]: https://godoc.org/k8s.io/kubernetes/cmd/kube-apiserver/app/options#ServerRunOptions "ServerRunOptions"
[4]: https://godoc.org/k8s.io/kubernetes/pkg/proxy/apis/config#KubeProxyConfiguration "KubeProxyConfiguration"
[5]: https://godoc.org/k8s.io/kubernetes/pkg/controller/apis/config#KubeControllerManagerConfiguration "KubeControllerManagerConfiguration"
[6]: https://godoc.org/github.com/coreos/etcd/etcdserver#ServerConfig "ServerConfig"

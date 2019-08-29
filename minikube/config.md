# Minikube 部署时，修改 kubernetes 的组件参数

minikube 支持修改每个组件的参数，用 --extra-config 中，格式如下：

```sh
$ ./minkube-v1.3.1 start --extra-config=scheduler.leader-elect-resource-lock=configmaps  --extra-config=controller-manager.leader-elect-resource-lock=configmaps
😄  minikube v1.3.1 on Darwin 10.14
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🏃  Using the running virtualbox "minikube" VM ...
⌛  Waiting for the host to be provisioned ...
🐳  Preparing Kubernetes v1.15.2 on Docker 18.09.8 ...
    ▪ scheduler.leader-elect-resource-lock=configmaps
    ▪ controller-manager.leader-elect-resource-lock=configmaps
🔄  Relaunching Kubernetes using kubeadm ...
⌛  Waiting for: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
```

上面参数的含义是将 scheduler 的参数 LeaderElection.ResourceLock 设置为 configmaps。

支持参数设置的组件：[kubelet][2]、[apiserver][3]、[proxy][4]、[controller-manager][5]、[etcd][6]、[scheduler][1]。

## 参考

[1]: https://godoc.org/k8s.io/kubernetes/pkg/scheduler/apis/config#KubeSchedulerConfiguration "KubeSchedulerConfiguration"
[2]: https://godoc.org/k8s.io/kubernetes/pkg/kubelet/apis/config#KubeletConfiguration "KubeletConfiguration"
[3]: https://godoc.org/k8s.io/kubernetes/cmd/kube-apiserver/app/options#ServerRunOptions "ServerRunOptions"
[4]: https://godoc.org/k8s.io/kubernetes/pkg/proxy/apis/config#KubeProxyConfiguration "KubeProxyConfiguration"
[5]: https://godoc.org/k8s.io/kubernetes/pkg/controller/apis/config#KubeControllerManagerConfiguration "KubeControllerManagerConfiguration"
[6]: https://godoc.org/github.com/coreos/etcd/etcdserver#ServerConfig "ServerConfig"

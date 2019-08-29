# Minikube 部署时，修改 kubernetes 的组件参数

minikube 支持修改每个组件的参数，用 --extra-config 中，格式如下：

```sh
./minkube start --extra-config=scheduler.LeaderElection.ResourceLock=configmaps
```

上面参数的含义是将 scheduler 的参数 LeaderElection.ResourceLock设置为 configmaps。

需要注意 “组件名.” 后面跟随的是 [KubeSchedulerConfiguration][1] 中的字段，不是组件的命令行参数。 scheduler.LeaderElection.ResourceLock 对应的命令行参数是 --leader-elect-resource-lock。

支持参数设置的组件：[kubelet][2]、[apiserver][3]、[proxy][4]、[controller-manager][5]、[etcd][6]、[scheduler][1]。点击连接就可以查看支持的配置项。


## 参考

[1]: https://godoc.org/k8s.io/kubernetes/pkg/scheduler/apis/config#KubeSchedulerConfiguration "KubeSchedulerConfiguration"
[2]: https://godoc.org/k8s.io/kubernetes/pkg/kubelet/apis/config#KubeletConfiguration "KubeletConfiguration"
[3]: https://godoc.org/k8s.io/kubernetes/cmd/kube-apiserver/app/options#ServerRunOptions "ServerRunOptions"
[4]: https://godoc.org/k8s.io/kubernetes/pkg/proxy/apis/config#KubeProxyConfiguration "KubeProxyConfiguration"
[5]: https://godoc.org/k8s.io/kubernetes/pkg/controller/apis/config#KubeControllerManagerConfiguration "KubeControllerManagerConfiguration"
[6]: https://godoc.org/github.com/coreos/etcd/etcdserver#ServerConfig "ServerConfig"

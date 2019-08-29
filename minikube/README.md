# Minikube 使用方法（最详细的中文手册）

[用 minikube 部署开发测试环境][1] 中记录了 minikube 的使用方法，这里重新整理一下。

做 Kubernetes 配套开发时，用 minkube 启动一个 kubernetes 集群作为开发测试环境是非常方便的。Minikube 可以方便地启动不同版本的 kubernetes，美中不足的是，用 minikube 部署的 kubernetes 都是单机模式，如果要做一些集群相关调试，单机的 kubernetes 就不能满足需求了。

Minikube 是 kubernetes 的一个子项目（[github地址][2]），使用手册比较完善， [minikube doc][3]。

Minkube 在 linux、windows 和 mac 中都可以使用，在 linux 可以使用 baremetal 的方式，将 kubernetes 直接部署在当前系统上，在 windows 和 mac 上都是创建一个虚拟机，在虚拟机中部署 kubernetes，虚拟机的创建都是自动的。

## 参考

[1]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/10/03/k8s-class-deploy.html "Kubernetes1.12从零开始（二）：用minikube部署开发测试环境"
[2]: https://github.com/kubernetes/minikube "github.com/kubernetes/minikube"
[3]: https://minikube.sigs.k8s.io/docs/start/ "minikube doc"

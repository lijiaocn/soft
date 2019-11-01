<!-- toc -->
# istio 的部署

在 Kubernetes 集群中部署 istio 是最方便的，参考 [Installing on Kubernetes][1]。

* 单纯了解学习，部署 [istio 预览版](./demo-install.md)
* 生产使用，部署 istio 生产版

无论采用哪种方式部署，都需要下载 istio 部署文件，准备一个 kubernetes 集群。部署文件到 github 下载，Kubernetes 集群可以使用 [minikube](../minikube/index.md) 部署一个单机版的 kubernetes。

本手册中部署的 istio 版本是 1.2.5：

	Istio 1.2 has been tested with these Kubernetes releases: 1.12, 1.13, 1.14.

## 准备 kubernetes 集群

单机版 Kubernetes 集群创建：[minkube 使用手册](../minikube/index.md)。

多机版 Kubernetes 集群参考（下面三份文档比较老，以后会更新）：

* [《Kubernetes1.12从零开始（三）：用kubeadm部署多节点集群》][5]
* [《Kubernetes1.12从零开始（五）：自己动手部署kubernetes》][6]
* [《Kubernetes1.12从零开始（六）：从代码编译到自动部署》][7]

## 下载 istio 部署文件

从 github 的 [release][3] 页面下载 istio，例如：

```sh
wget https://github.com/istio/istio/releases/download/1.2.5/istio-1.2.5-linux.tar.gz
```

本手册的操作环境是 mac + minikube，下载的是 istio-1.2.5-osx.tar.gz：

```sh
wget https://github.com/istio/istio/releases/download/1.2.5/istio-1.2.5-osx.tar.gz
```

解压后得到 istio-1.2.5，里面有安装文件和操作命令（bin/istioctl）：

```sh
$ tree -L 1 istio-1.2.5
istio-1.2.5
├── LICENSE
├── README.md
├── bin
├── install
├── istio.VERSION
├── samples
└── tools
```

或者直接 clone 整个 istio 项目，然后切换到要使用的版本，这种方式需要自己编译 istio，编写部署文件：

```sh
git clone https://github.com/istio/istio.git
git checkout 1.2.5 -b 1.2.5
```

源码目录 [install/kubernetes][4] 中提供了一些部署文件：

```sh
➜  istio git:(1.2.5) ✗ tree install/kubernetes -L 3
install/kubernetes
├── README.md
├── ansible
│   ├── OWNERS
│   ├── README.md
│   ├── ansible.cfg
│   ├── istio
│   │   ├── defaults
│   │   ├── meta
│   │   ├── tasks
│   │   └── vars
│   └── main.yml
├── global-default-sidecar-scope.yaml
├── helm
│   ├── README.md
│   ├── helm-service-account.yaml
│   ├── istio
│   │   ├── Chart.yaml
│   │   ├── LICENSE
│   │   ├── README.md
│   │   ├── charts
│   │   ├── example-values
│   │   ├── files
│   │   ├── requirements.yaml
│   │   ├── templates
│   │   ├── test-values
│   │   ├── values-istio-demo-auth.yaml
│   │   ├── values-istio-demo-common.yaml
│   │   ├── values-istio-demo.yaml
│   │   ├── values-istio-minimal.yaml
│   │   ├── values-istio-remote.yaml
│   │   ├── values-istio-sds-auth.yaml
│   │   └── values.yaml
│   └── istio-init
│       ├── Chart.yaml
│       ├── LICENSE
│       ├── README.md
│       ├── files
│       ├── templates
│       └── values.yaml
├── mesh-expansion.yaml
└── namespace.yaml
```

部署操作见后面的章节。

## 参考

[1]: https://istio.io/docs/setup/kubernetes/ "Installing on Kubernetes"
[2]: https://istio.io/docs/setup/kubernetes/install/kubernetes/ "Quick Start Evaluation Install"
[3]: https://github.com/istio/istio/releases "istio/releases"
[4]: https://github.com/istio/istio/tree/1.2.5/install/kubernetes "1.2.5/install/kubernetes "
[5]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/10/04/k8s-class-a-deploy-kubeadm.html "《Kubernetes1.12从零开始（三）：用kubeadm部署多节点集群》"
[6]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/10/07/k8s-class-deploy-from-scratch.html "《Kubernetes1.12从零开始（五）：自己动手部署kubernetes》"
[7]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/11/04/k8s-class-build-and-deploy-by-ansible.html "《Kubernetes1.12从零开始（六）：从代码编译到自动部署》"

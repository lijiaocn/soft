<!-- toc -->
# Kubernetes 源代码阅读指引

## 独立发布的代码

kubernetes/staging/ 位于 kubernetes 项目中，但是其中的代码发布到独立的 repo：

* [k8s.io 是怎么回事？](https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2019/12/06/k8s-io-usage.html)
* [kubernetes 调度组件 kube-scheduler 1.16.3 源代码阅读指引](https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2019/12/08/kube-scheduler-code-1-16-3.html)

## api 定义代码

api 定义代码位于 [kubernetes/staging/src/k8s.io/api][2]。

需要注意 kubernetes 主项目中还有三个名为 api/apis 的目录：

```sh
kubernetes/api:      不符合规范、但是还没去处的 api 列表
kuberntes/pkg/api:   简单的操作函数
kuberntes/pkg/apis:  deepcopy、版本转换等工具代码
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://github.com/kubernetes/kubernetes/tree/v1.16.3/staging/src/k8s.io/api "k8s.io/api"

# Istio 的应用示例拆解

Istio 的代码目录和 release 文件中提供多个 [应用示例][1]：

```sh
$ tree -L 1 istio-1.2.5-linux/samples
istio-1.2.5-linux/samples
├── README.md
├── bookinfo
├── certs
├── custom-bootstrap
├── external
├── fortio
├── health-check
├── helloworld
├── httpbin
├── https
├── kubernetes-blog
├── rawvm
├── sleep
├── tcp-echo
└── websockets
```

[examples][2] 中有简单的介绍。

## 参考

[1]: https://github.com/istio/istio/tree/master/samples  "istio samples"
[2]: https://istio.io/docs/examples/ "examples"

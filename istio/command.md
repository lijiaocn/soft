<!-- toc -->
# istio 操作命令

istio 的主要操作命令是 [istioctl][2]，命令文件位于下载的安装包中：

```sh
$ ls istio-1.4.0/bin
istioctl
```

## istio 提供的 Web 页面

istio 内置了一些 web 页面，可以用 `istioctl dashboard` 命令打开。 

```sh
$ ./istioctl help dashboard
Access to Istio web UIs

Usage:
  istioctl dashboard [flags]
  istioctl dashboard [command]

Aliases:
  dashboard, dash, d

Available Commands:
  kiali       Open Kiali web UI
  controlz    Open ControlZ web UI
  envoy       Open Envoy admin web UI
  jaeger      Open Jaeger web UI
  prometheus  Open Prometheus web UI
  grafana     Open Grafana web UI
  zipkin      Open Zipkin web UI
```

### kiali，可视化页面

kiali 是 istio 内置的可视化页面：

```sh
$ ./istioctl d kiali
http://localhost:50005/kiali
```

![istio kiali 的页面](../img/istio/kiali.png)

### controlz，配置文件

打开配置页面，istio 的配置用 pilot 组件管理的，打开配置页面时，需要指定 pilot 容器：

```sh
$ ./istioctl dashboard  controlz istio-pilot-785bc88559-jjzvl -n istio-system
```

![istio configz 的页面](../img/istio/configz.png)

### envoy，指定 envoy 的管理页面

istio 为网格内的每个容器注入了一个 envoy 容器，同时提供打开每个 envoy 的 admin 页面方法：

```sh
$ ./istioctl d envoy  ratings-v1-779cf974b6-fxmk8
http://localhost:51927
```

我们可以通过 envoy 的 admin 页面查看 envoy 中规则，envoy 用法参考 [envoy 使用手册](../envoy/index.md)。

![istio envoy 的页面](../img/istio/envoy.png)

### jaejer，调用链

istio 支持 jaejer 和 zipkin。

```sh
$ ./istioctl d jaeger
http://localhost:50887
```

![istio jaeger 的页面](../img/istio/jaeger.png)

### prometheus，监控数据

打开 prometheus 页面：

```sh
$ ./istioctl d prometheus
http://localhost:49226
```

![istio prometheus 的页面](../img/istio/prometheus.png)

### grafana，监控数据

打开 grafana 页面：

```sh
$ ./istioctl d grafana
http://localhost:49645
```

![istio grafana 的页面](../img/istio/grafana.png)

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://istio.io/docs/reference/commands/istioctl/ "istioctl"

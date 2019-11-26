<!-- toc -->
# istio 服务运行指标的采集和查看

服务的运行指标要先配置然后才能生成。首先在 istio 中定义一个指标、然后定义指标的收集方式，最后定义指标的收集规则，之后就可以在 istio 中看到采集的指标，参考 [Collecting Metrics][2]。

下面操作在 [Bookinfo Application](./bookinfo.md) 的基础上进行。

## 创建指标并收集

metrics.yaml 包含了所有需要的配置：

```sh
$ kubectl apply -f samples/bookinfo/telemetry/metrics.yaml
```

### 指标定义

定义名为 doublerequestcount 的指标，instance 规定了指标的生成方法，dimensions 定义了指标的属性和属性值（属性值中可以引用 envoy 和 mixer 生成的属性） ：

```yaml
# Configuration for metric instances
apiVersion: config.istio.io/v1alpha2
kind: instance
metadata:
  name: doublerequestcount
  namespace: istio-system
spec:
  compiledTemplate: metric
  params:
    value: "2" # count each request twice
    dimensions:
      reporter: conditional((context.reporter.kind | "inbound") == "outbound", "client", "server")
      source: source.workload.name | "unknown"
      destination: destination.workload.name | "unknown"
      message: '"twice the fun!"'
    monitored_resource_type: '"UNSPECIFIED"'
```

doublerequestcount 有个四个属性：reporter、source、destination 和 message。

value 是指标的值，这里固定为 2，每次请求都会生成一个带有四个属性、值为 2 的指标。

### 收集器定义

收集器定义指标的存放位置和需要存放的属性：

```yaml
# Configuration for a Prometheus handler
apiVersion: config.istio.io/v1alpha2
kind: handler
metadata:
  name: doublehandler
  namespace: istio-system
spec:
  compiledAdapter: prometheus
  params:
    metrics:
    - name: double_request_count # Prometheus metric name
      instance_name: doublerequestcount.instance.istio-system # Mixer instance name (fully-qualified)
      kind: COUNTER
      label_names:
      - reporter
      - source
      - destination
      - message
```

doublehandler 表示在 prometheus 中存放 doublerequestcount 指标时，把指标的四个属性作为设置为 prometheus 中的 label，同时 doublerequestcount 的指标值被累加（ COUNTER 类型）。这样一来 doublerequestcount 的含义就是已经发生的请求次数

### 采集规则定义

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://istio.io/docs/tasks/observability/metrics/collecting-metrics/ "Collecting Metrics"

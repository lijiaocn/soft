<!-- toc -->
# istio 的日志收集方法

istio 为每个 pod 注入 envoy 容器后，envoy 能够记录每个 pod 的请求。

## 原始访问日志

是否记录访问日志以及访问日志的存放位置，通过 [accessLogFile][4] 设置：

```yaml
# $ kubectl -n istio-system get configmap  istio  -o yaml |grep accessLogFile
# Set accessLogFile to empty string to disable access log.
accessLogFile: "/dev/stdout"
```

默认显示到标准输出，可以用下面的方式查看：

```sh
$ kubectl logs -f productpage-v1-667bc85676-sgbqp -c istio-proxy
[2019-11-27T06:45:55.152Z] "GET /reviews/0 HTTP/1.1" 200 - "-" "-" 0 379 74 74 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 Safari/537.36" "105f2059-fbc8-9446-8bbb-b9c1a37019bd" "reviews:9080" "172.17.0.32:9080" outbound|9080|v2|reviews.default.svc.cluster.local - 10.100.249.106:9080 172.17.0.27:40612 - -
[2019-11-27T06:45:55.121Z] "GET /productpage HTTP/1.1" 200 - "-" "-" 0 5286 110 103 "172.17.0.1" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 Safari/537.36" "105f2059-fbc8-9446-8bbb-b9c1a37019bd" "192.168.99.100:31380" "127.0.0.1:9080" inbound|9080|http|productpage.default.svc.cluster.local - 172.17.0.27:9080 172.17.0.1:0 - default
...
```

## 生成并收集日志

istio 支持自定义日志格式，并将统一收集到指定位置。这一步操作和 [指标采集](./metrics.md) 类似，通过 instance、handler 和 route 三个 CRD 设置。

[log-entry.yaml][5] 定义了日志 newlog，并收集到 mixer 的标准输出：

```sh
$ kubectl apply -f samples/bookinfo/telemetry/log-entry.yaml
```

instance 定义的日志格式，名称为 newlog，在 params 中设置了级别、时间戳以及属性组成：

```yaml
# Configuration for logentry instances
apiVersion: config.istio.io/v1alpha2
kind: instance
metadata:
  name: newlog
  namespace: istio-system
spec:
  compiledTemplate: logentry
  params:
    severity: '"warning"'
    timestamp: request.time
    variables:
      source: source.labels["app"] | source.workload.name | "unknown"
      user: source.user | "unknown"
      destination: destination.labels["app"] | destination.workload.name | "unknown"
      responseCode: response.code | 0
      responseSize: response.size | 0
      latency: response.duration | "0ms"
    monitored_resource_type: '"UNSPECIFIED"'
```

handler 定义了采集器，名称为 newloghandler，直接打印到 stdio，输出格式为 json：

```yaml
# Configuration for a stdio handler
apiVersion: config.istio.io/v1alpha2
kind: handler
metadata:
  name: newloghandler
  namespace: istio-system
spec:
  compiledAdapter: stdio
  params:
    severity_levels:
      warning: 1 # Params.Level.WARNING
    outputAsJson: true
```

rule 定义了采集动作，将 newlog 日志收集到 newloghandler：

```yaml
# Rule to send logentry instances to a stdio handler
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: newlogstdio
  namespace: istio-system
spec:
  match: "true" # match for all requests
  actions:
   - handler: newloghandler
     instances:
     - newlog
```

## 采集效果

上面定义的 newlog 采集规则，将日志输出到标准输出，这里的标准输出是 telemetry 的标准输出：

```sh
$ kubectl logs -f -n istio-system -l istio-mixer-type=telemetry -c mixer 
{"level":"warn","time":"2019-11-27T08:24:05.706343Z","instance":"newlog.instance.istio-system","destination":"productpage","latency":"1.072931146s","responseCode":200,"responseSize":4183,"source":"istio-ingressgateway","user":"unknown"}
{"level":"warn","time":"2019-11-27T08:24:05.188835Z","instance":"newlog.instance.istio-system","destination":"productpage","latency":"1.654626443s","responseCode":200,"responseSize":4183,"source":"istio-ingressgateway","user":"unknown"}
```

## 将日志收集到 es

handler 可以指向多种日志收集系统，例如 [通过flutentd收集][3]：

```yaml
# Configuration for a Fluentd handler
apiVersion: config.istio.io/v1alpha2
kind: handler
metadata:
  name: handler
  namespace: istio-system
spec:
  compiledAdapter: fluentd
  params:
    address: "fluentd-es.logging:24224"
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://istio.io/docs/tasks/observability/logs/access-log/ "Getting Envoy's Access Logs"
[3]: https://istio.io/docs/tasks/observability/logs/fluentd/ "Logging with Fluentd"
[4]: https://istio.io/docs/tasks/observability/logs/access-log/ "access-log"
[5]: https://istio.io/docs/tasks/observability/logs/collecting-logs/ "Collecting Logs"

<!-- toc -->
# istio 的流量策略（负载均衡策略）

istio 的流量策略在 [DestinationRule](./dstrule.md) 的 [TrafficPolicy][3] 中设置，负载均衡是流量策略中的一种。

下面操作在 [Bookinfo Application](./bookinfo.md) 的基础上进行。

## 断路保护

istio 的 [Circuit Breaking][2] 目前（2019-11-21 19:26:57）相当弱，就是控制一下请求数和并发数，感觉没有 [ingress-nginx 的限速功能](https://www.lijiaocn.com/soft/k8s/ingress-nginx/ratelimit.html) 实用。

### 限制 TCP 连接数和单连接请求数

编辑名为 details 的 DestinationRule，添加 trafficPolicy：

```sh
$ kubectl edit destinationrules details
...省略...
  host: details
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
      tcp:
        maxConnections: 1
...省略...
```

部署 istio 提供的 [fortio client][4]：

```sh
$ kubectl apply -f samples/httpbin/sample-client/fortio-deploy.yaml
```

4 并发测试，会发现部分请求被拒绝（503 错误）：

```sh
$ FORTIO_POD=$(kubectl get pod | grep fortio | awk '{ print $1 }')
$ kubectl exec -it $FORTIO_POD  -c fortio /usr/bin/fortio --  load   -c 4 -qps 0 -n 20 -loglevel Warning  http://details:9080/details/0
...省略...
Sockets used: 14 (for perfect keepalive, would be 4)
Code 200 : 8 (40.0 %)
Code 503 : 12 (60.0 %)
Response Header Sizes : count 20 avg 63.45 +/- 77.71 min 0 max 159 sum 1269
Response Body/Total Sizes : count 20 avg 281 +/- 46.03 min 241 max 337 sum 5620
All done 20 calls (plus 0 warmup) 39.719 ms avg, 87.7 qps
```

## 更多配置

* [TrafficPolicy][3]

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://istio.io/docs/tasks/traffic-management/circuit-breaking/ "Circuit Breaking"
[3]: https://istio.io/docs/reference/config/networking/destination-rule/#TrafficPolicy "TrafficPolicy"
[4]: https://istio.io/docs/tasks/traffic-management/circuit-breaking/#adding-a-client "fortio client"

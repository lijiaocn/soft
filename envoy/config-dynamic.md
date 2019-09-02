<!-- toc -->
# Envoy 的动态配置

Listener、Cluster 等除了在配置文件中静态配置，还可以从控制平面动态获取。下发动态配置的服务叫做 **控制平面** ，它的地址以 cluster 的形式在配置文件中静态配置，然后在 dynamic_resources 中引用这个 cluster。

可以动态下发的配置见 [Dynamic configuration][8]，主要有：

CDS：cluster 配置动态下发

LDS：listener 配置动态下发

EDS：cluster 中的 endpoint 的动态下发

RDS：listener 中的 route 的动态下发

SDS：证书的动态下发

其中只有 CDS 和 LDS 的发现地址是在 dynamic_resources 中指定的，后面有详细说明。

## 准备一个控制平面

[xds/xds.go][9] 是一个非常简陋的控制平面的实现，它的代码实现在 [控制平面实现](./control.md) 章节详细介绍，在这里我们只需要知道它既可以作为 xds 也可以作为 ads。

这个简陋的控制平面的监听地址是 0.0.0.0:5678，它只下发了下面这个 envoy 的配置：

```go
node := &core.Node{
	Id:      "envoy-64.58",
	Cluster: "test",
}
```

配置文件中使用同样的 ID 和 Cluster 的 envoy 才会接受这个控制平面下发的配置，这里使用的 envoy 配置文件中都使用这个 Id 和 Cluster：

```yaml
node:
  id: "envoy-64.58"
  cluster: "test"
```

编译运行：

```sh
git clone https://github.com/introclass/go-code-example.git
cd go-code-example/envoydev/xds/
$ ./xds
Enter to update version 1: Cluster_With_Static_Endpoint
<键入回车符，就开始下发所示的配置>
```

## 参考

[1]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/secret.html  "Secret discovery service (SDS)"
[2]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/http_conn_man/rds.html "Route discovery service (RDS)"
[3]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/listeners/lds.html  "LDS"
[4]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/cluster_manager/cds.html "CDS"
[5]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/overview/v2_overview#aggregated-discovery-service  "ADS"
[6]: https://www.envoyproxy.io/docs/envoy/v1.11.0/intro/arch_overview/operations/dynamic_configuration#arch-overview-dynamic-config-eds "EDS"
[7]: https://www.envoyproxy.io/docs/envoy/v1.11.0/api-docs/xds_protocol#eventual-consistency-considerations "xDS REST and gRPC protocol"
[8]: https://www.envoyproxy.io/docs/envoy/v1.11.0/intro/arch_overview/operations/dynamic_configuration  "Dynamic configuration"
[9]: https://github.com/introclass/go-code-example/blob/master/envoydev/xds/xds.go "xds/xds.go"

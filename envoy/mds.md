<!-- toc -->
# envoy 的 lds/cds/rds/sds/eds

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

dynamic_resources 配置中只有 [lds_config][3]、[cds_config][4] 和 [ads_config][5]，分别对应 listenter、cluster 和聚合发现：

```json
"dynamic_resources": {
  "lds_config": "{...}",
  "cds_config": "{...}",
  "ads_config": "{...}"
},
```

但可以动态下发的配置不只有 listener 和 cluster。

cluster 中的 endpoint、tls 用到的 secret、HttpConnectionManager 中用到 route 也可以动态下发，对应的发现服务分别是 [eds][6]、[sds][1]、[rds][2]。这些发现服务不在 dynamic_resources 中配置，而是独立配置或者在用到它们的 filter 中配置。

## RDS 的配置

以 HttpConnectionManager 为例。

HttpConnectionManager 是一个处理 http 请求的 filter，它用到的 http 路由可以从 rds 中动态发现，rds 就在这个 filter 中指定。

下面是我们使用的简陋控制平面中一段代码，可以看到 `Rds` 字段配置的类型是 Ads，http 路由将从 ads_cluster 中获取：

```go
listen_filter_http_conn_ := &http_conn_manager.HttpConnectionManager{
    StatPrefix: "ingress_http",
    RouteSpecifier: &http_conn_manager.HttpConnectionManager_Rds{
        Rds: &http_conn_manager.Rds{
            RouteConfigName: "ads_route",
            ConfigSource: &core.ConfigSource{
                ConfigSourceSpecifier: &core.ConfigSource_Ads{
                    Ads: &core.AggregatedConfigSource{}, //使用ADS
                },
            },
        },
    },
    ...
}
```

类似于 ADS 方式中的 cds_config 和 lds_config：

```yaml
dynamic_resources:
  ads_config:
    api_type: GRPC
    grpc_services:
      envoy_grpc:
        cluster_name: ads_cluster
  cds_config: {ads: {}}
  lds_config: {ads: {}}
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

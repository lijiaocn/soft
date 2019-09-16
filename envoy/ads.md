<!-- toc -->

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

# envoy 用 ADS 动态发现配置的方法



这里使用的配置文件是 [envoy-docker-run/envoy-1-ads.yaml][9]。

## 为什么要使用 ADS ？

cds_config 和 lds_config 是分开的，这意味着 cluster 和 listener 配置可以从不同的控制平面获取，这样会遇到配置不同步的问题，即使它们用的是同一个控制平面也可能因为到达次序不同而不同步。

譬如 listenerA 中使用了 clusterA，但是 listenerA 的配置可能在 clusterA 之前下发到 envoy ，使 envoy 认为 listenerA 使用了一个不存在的 clusterA。

[ADS][5] 支持所有类型动态配置的下发，并且会处理配置间的依赖，保证配置的下发顺序，是优先选用的配置发现方法，实现原理见配置下发协议的说明 [xDS REST and gRPC protocol][7]。

## 配置 ADS 地址

首先配置 ADS 的地址，ADS 地址的配置方式与 XDS 相同，都是以 cluster 形式在配置文件中设置：

```yaml
static_resources:
  clusters:
  - name: ads_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    load_assignment:
      cluster_name: ads_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 5678
```

## 配置 ads_config

然后配置 ads_config，在 ads_config 中引用上面配置的 ads_cluster，另外 cds_config 和  lds_config 设置成 ads 方式，如下：

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

## 运行效果

除了启动 envoy 时用 envoy-1-ads.yaml 文件，过程与 [用 XDS 下发配置](./xds.md#运行效果) 相同：

```sh
./run.sh envoy-1-ads.yaml
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
[9]: https://github.com/introclass/go-code-example/blob/master/envoydev/xds/envoy-docker-run/envoy-1-ads.yaml "envoy-1-ads.yaml"

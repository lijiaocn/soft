<!-- toc -->
# Envoy 的动态配置示例

Listener、Cluster 等除了在配置文件中静态配置，还可以从控制平面动态获取，动态配置在 `dynamic_resources` 中设置。但是注意，提供动态配置的服务地址需要在文件中静态配置，动态配置里引用静态配置中的配置发现服务。配置发现服务通常称为 **控制平面**。

要注意区分下发动态配置的控制平面和 envoy 依赖的外部服务，envoy 一些功能依赖于外部的服务，譬如认证、全局限速等，这些功能在运行中需要请求外部的认证服务和限速服务，因此需要为其配置外部服务地址。这些外部服务和下发 envoy 配置的控制平面的用途是不同的，建议分开看待。

动态配置通常是指控制平面向 envoy 下发的配置，可以动态下发的配置见：[Dynamic configuration][8]。

## cds_config、lds_config：动态发现 cluster 和 listener

在文件中静态配置控制平面的地址，然后在 dynamic_resources 中为 cds_config（cluster 动态发现） 和 lds_config（listener 动态发现），指定控制平面：

```yaml
node:
  id: "envoy-64.58"
  cluster: "test"
#runtime:
#  symlink_root: /srv/runtime/current
#  subdirectory: envoy
#  override_subdirectory: envoy_override
watchdog:
  miss_timeout: 0.2s
  megamiss_timeout: 1s
  kill_timeout: 0s
  multikill_timeout: 0s
flags_path: /etc/envoy/flags/
stats_flush_interval: 5s
stats_config:
  use_all_default_tags: true
stats_sinks:
  name: envoy.stat_sinks.hystrix
  config:
    num_buckets: 10
admin:
  access_log_path: /var/log/envoy/admin_access.log
  profile_path: /var/log/envoy/envoy.prof
  address:
    socket_address:
      protocol: TCP
      address: 0.0.0.0
      port_value: 9901
dynamic_resources:
  cds_config:
    api_config_source:
      api_type: GRPC
      grpc_services:
        envoy_grpc:
          cluster_name: xds_cluster
  lds_config:
    api_config_source:
      api_type: GRPC
      grpc_services:
        envoy_grpc:
          cluster_name: xds_cluster
static_resources:
  clusters:
  - name: xds_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    load_assignment:
      cluster_name: xds_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 5678
```

## ads_config：发现所有类型的动态配置

cds_config 和 lds_config 分开设置，意味着 cluster 和 listener 配置可以从不同的控制平面获取，会出现配置不同步的问题，即使两者使用同一个控制平面也可能因为到达次序不同而不同步。譬如 listenerA 中使用了 clusterA，但是 listenerA 的配置可能在 clusterA 之前下发到 envoy ，使 envoy 认为 listenerA 使用了一个不存在的 clusterA。

[ADS][5] 支持所有类型动态配置的下发，并且会处理配置间的依赖，保证配置的下发顺序，是优先选用的配置发现方法，实现原理见配置下发协议的说明 [xDS REST and gRPC protocol][7]。

使用 ADS 时，cds_config 和 lds_config 以及其其它需要填充地址的动态配置，只需要声明用ads 即可：

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

一个使用 ads 的配置：
 
```yaml
node:
  id: "envoy-64.58"
  cluster: "test"
#runtime:
#  symlink_root: /srv/runtime/current
#  subdirectory: envoy
#  override_subdirectory: envoy_override
watchdog:
  miss_timeout: 0.2s
  megamiss_timeout: 1s
  kill_timeout: 0s
  multikill_timeout: 0s
flags_path: /etc/envoy/flags/
stats_flush_interval: 5s
stats_config:
  use_all_default_tags: true
stats_sinks:
  name: envoy.stat_sinks.hystrix
  config:
    num_buckets: 10
admin:
  access_log_path: /var/log/envoy/admin_access.log
  profile_path: /var/log/envoy/envoy.prof
  address:
    socket_address:
      protocol: TCP
      address: 0.0.0.0
      port_value: 9901
dynamic_resources:
  ads_config:
    api_type: GRPC
    grpc_services:
      envoy_grpc:
        cluster_name: ads_cluster
  cds_config: {ads: {}}
  lds_config: {ads: {}}
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
  - name: xds_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    load_assignment:
      cluster_name: xds_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 5678
```

## lds/cds/rds/sds/eds

dynamic_resources 配置中只有 [lds_config][3]、[cds_config][4] 和 [ads_config][5]：

```json
"dynamic_resources": {
  "lds_config": "{...}",
  "cds_config": "{...}",
  "ads_config": "{...}"
},
```

可以动态下发的配置不只有 listener 和 cluster，cluster 中的 endpoint、tls 用到 secret、HttpConnectionManager 中用到 route 也可以动态下发，分别被称为 [eds][6]、[sds][1]、[rds][2]，它们的出现位置 dynamic_resources 中，而是分散在各自的定义位置中。

以 HttpConnectionManager 为例，动态下发的 listener 中使用了 HttpConnectionManager，并在下发的动态配置中，指定 HttpConnectionManager 使用 rds，下面是用 go-controller-plane 下发配置的代码中一段，指定 rds 使用 ads：

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
    HttpFilters: httpFilters,
}
```

## 参考

[1]: https://www.envoyproxy.io/docs/envoy/latest/configuration/secret.html  "Secret discovery service (SDS)"
[2]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http_conn_man/rds.html "Route discovery service (RDS)"
[3]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/lds.html  "LDS"
[4]: https://www.envoyproxy.io/docs/envoy/latest/configuration/cluster_manager/cds.html "CDS"
[5]: https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/v2_overview#aggregated-discovery-service  "ADS"
[6]: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/dynamic_configuration#arch-overview-dynamic-config-eds "EDS"
[7]: https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol#eventual-consistency-considerations "xDS REST and gRPC protocol"
[8]: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/dynamic_configuration#arch-overview-dynamic-config-eds  "Dynamic configuration"

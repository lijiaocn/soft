<!-- toc -->
# Envoy 的动态配置示例

Listner、Cluster 等除了在配置文件中静态配置，还可以从控制平面动态获取。

## cds_config、lds_config

控制平面的服务地址在配置文件静态配置，然后在 dynamic_resources 中为 cds_config（cluster 动态发现） 和 lds_config（listener 动态发现），指定控制平面：

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

## ads_config

cds_config 和 lds_config 分配设置，这意味着 cluster 和 listener 可以从不同的控制的平面获取，但是配置的到达顺序不同，可能带来一些问题。譬如 listenerA 中使用了 clusterA，但是 listenerA 可能在 clusterA 之前下发到 envoy ，使 envoy 认为 listenerA 使用了一个不存在的 clusterA。

使用支持所有类型配置下发的 [ADS][5] 可以避免这个问题：


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

但可以动态下发的配置不只有 listener 和 cluster，HttpConnectionManager 中用到 route 、tls 用到 secret、cluster 中的 endpoint 也可以动态下发，分别被称为 [rds][2]、[sds][1]、[eds][6]，它们不是在 dynamic_resources 中指定的，而是用到的地方指定。 

[1]: https://www.envoyproxy.io/docs/envoy/latest/configuration/secret.html  "Secret discovery service (SDS)"
[2]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http_conn_man/rds.html "Route discovery service (RDS)"
[3]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/lds.html  "LDS"
[4]: https://www.envoyproxy.io/docs/envoy/latest/configuration/cluster_manager/cds.html "CDS"
[5]: https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/v2_overview#aggregated-discovery-service  "ADS"
[6]: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/dynamic_configuration#arch-overview-dynamic-config-eds "EDS"

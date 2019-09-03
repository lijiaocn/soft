# Envoy 的 Cluster 配置详解

[Clusters][1] 相当于 nginx 的 upstream，是一组 IP 或者域名的集合， 是 Envoy 收到的请求最终流向的地方。 Cluster 的配置项比较多，见 [Clusters Config][2]：

```json
{
  "name": "...",
  "alt_stat_name": "...",
  "type": "...",
  "cluster_type": "{...}",
  "eds_cluster_config": "{...}",
  "connect_timeout": "{...}",
  "per_connection_buffer_limit_bytes": "{...}",
  "lb_policy": "...",
  "hosts": [],
  "load_assignment": "{...}",
  "health_checks": [],
  "max_requests_per_connection": "{...}",
  "circuit_breakers": "{...}",
  "tls_context": "{...}",
  "common_http_protocol_options": "{...}",
  "http_protocol_options": "{...}",
  "http2_protocol_options": "{...}",
  "extension_protocol_options": "{...}",
  "typed_extension_protocol_options": "{...}",
  "dns_refresh_rate": "{...}",
  "respect_dns_ttl": "...",
  "dns_lookup_family": "...",
  "dns_resolvers": [],
  "outlier_detection": "{...}",
  "cleanup_interval": "{...}",
  "upstream_bind_config": "{...}",
  "lb_subset_config": "{...}",
  "ring_hash_lb_config": "{...}",
  "original_dst_lb_config": "{...}",
  "least_request_lb_config": "{...}",
  "common_lb_config": "{...}",
  "transport_socket": "{...}",
  "metadata": "{...}",
  "protocol_selection": "...",
  "upstream_connection_options": "{...}",
  "close_connections_on_host_health_failure": "...",
  "drain_connections_on_host_removal": "...",
  "filters": []
}
```

## Cluster 的类型

type 和 cluster_type 是 cluster 的类型，type 是 envoy 支持的标准类型，cluster_type 是自定义类型。

envoy 支持的标准类型按照 [service discovery types][3] 分类：

* Static，直接配置 IP 地址
* Strict DNS，通过解析域名获取目标 IP，使用查询出来的所有 IP
* Logical DNS，通过解析域名获取目标 IP，只使用第一个 IP 
* Endpoint discovery service（EDS）
* Original destination，透明代理方式

在 go-controller-plane 中的定义分别是：

```go
// envoy/api/v2/cds.pb.go: 43
const (
	Cluster_STATIC Cluster_DiscoveryType = 0
	Cluster_STRICT_DNS Cluster_DiscoveryType = 1
	Cluster_LOGICAL_DNS Cluster_DiscoveryType = 2
	Cluster_EDS Cluster_DiscoveryType = 3
	Cluster_ORIGINAL_DST Cluster_DiscoveryType = 4
)
```

在配置文件中的写法分别是：

```go
// envoy/api/v2/cds.pb.go: 72
var Cluster_DiscoveryType_value = map[string]int32{
	"STATIC":       0,
	"STRICT_DNS":   1,
	"LOGICAL_DNS":  2,
	"EDS":          3,
	"ORIGINAL_DST": 4,
}
```

## Cluster 的 filter

Cluster 的 filter 字段中是 [network filter](./network-filter.md)，处理从 cluster 流出（outgoing）的数据。

## Cluster 的负载均衡算法

Cluster 支持下面的 [负载均衡算法][4]：

* ROUND_ROBIN
* LEAST_REQUEST
* RING_HASH
* RANDOM
* MAGLEV
* CLUSTER_PROVIDED

## 参考

[1]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/clusters/clusters "Envoy Clusters"
[2]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto#cluster "Envoy Clusters Config"
[3]: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/service_discovery "Supported service discovery types"
[4]: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/load_balancing/load_balancers#arch-overview-load-balancing-types "Supported load balancers"

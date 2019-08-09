# go-control-plane 中的配置定义

[go-control-plane](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/29/envoy-07-features-2-dynamic-discovery.html#go-control-plane) 中的配置定义与 [API 文档](https://www.envoyproxy.io/docs/envoy/latest/api/api) 中给出的相同。以 listener 为例，[listeners][12] 在 api 文档中定义：

```json
{
  "name": "...",
  "address": "{...}",
  "filter_chains": [],
  "use_original_dst": "{...}",
  "per_connection_buffer_limit_bytes": "{...}",
  "metadata": "{...}",
  "drain_type": "...",
  "listener_filters": [],
  "listener_filters_timeout": "{...}",
  "transparent": "{...}",
  "freebind": "{...}",
  "socket_options": [],
  "tcp_fast_open_queue_length": "{...}",
  "traffic_direction": "..."
}
```

go-control-plane 在 [envoy/api/v2/lds.pb.go](https://github.com/envoyproxy/go-control-plane/blob/v0.8.4/envoy/api/v2/lds.pb.go) 中实现的 Listener：

```go
type Listener struct {
    Name string 
    Address *core.Address 
    FilterChains []*listener.FilterChain 
    UseOriginalDst *types.BoolValue  
    PerConnectionBufferLimitBytes *types.UInt32Value 
    Metadata *core.Metadata 
    DeprecatedV1 *Listener_DeprecatedV1 
    DrainType Listener_DrainType 
    ListenerFilters []*listener.ListenerFilter 
    ListenerFiltersTimeout *time.Duration 
    Transparent *types.BoolValue 
    Freebind *types.BoolValue 
    SocketOptions []*core.SocketOption 
    TcpFastOpenQueueLength *types.UInt32Value 
    XXX_NoUnkeyedLiteral   struct{}           
    XXX_unrecognized       []byte             
    XXX_sizecache          int32              
}
```



## 参考

[1]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto#envoy-api-msg-config-filter-network-http-connection-manager-v2-httpconnectionmanager  "HTTP Connection Manager"
[2]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/network "network filter"
[3]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/route/route.proto#route-virtualhost  "route-virtualhost"
[4]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/rds.proto#envoy-api-msg-routeconfiguration "routeconfiguration"
[5]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/route/route.proto "route.proto"
[6]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto#cluster "cluster"
[7]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/endpoint/endpoint.proto  "endpoint.proto"
[8]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#listener "listener"
[9]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/auth/cert.proto#envoy-api-msg-auth-secret "secret"
[10]: https://www.envoyproxy.io/docs/envoy/latest/configuration/cluster_manager/cds.html "CDS"
[11]: https://www.envoyproxy.io/docs/envoy/latest/configuration/secret.html  "SDS"
[12]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#listener "listener"
[13]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/lds.html  "LDS"
[14]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http_conn_man/rds.html "Route discovery service (RDS)"
[15]: https://github.com/envoyproxy/data-plane-api  "envoyproxy/data-plane-api"
[16]: https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol  "xDS REST and gRPC protocol"
[17]: https://github.com/envoyproxy/go-control-plane  "envoyproxy/go-control-plane"

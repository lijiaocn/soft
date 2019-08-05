# Envoy 的动态配置（配置规则下发）

Envoy 的一些配置项可以动态下发，envoy 实例启动时从指定的配置文件中获取下发动态配置的服务地址，然后从通过这些事先配好的服务，实时获取动态配置。

可以动态下发的配置有：

* [cluster][6]
* [cluster][6] 中的 [endpoint][7]
* [listener][8]
* listener 的 [Network filter][2] 中的 [HTTP connection manager][1] 中的 [Virtualhost][3] 中的 [RouteConfiguration][4] 中的 [route][5]
* [serect][9]

对应的服务端分别是 [cds][10]、[lds][10]、[sds][11] 和可以同时提供所有动态配置的 [ads][10]

当配置项有依赖关系时，ADS 可以保障将依赖的配置项一同下发，没有特殊情况就直接用 ADS。

## 动态下发的配置格式

通过控制平面下发的规则和配置文件中对应的配置项格式是一致的。

通过 [go-control-plane](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/29/envoy-07-features-2-dynamic-discovery.html#go-control-plane) 下发规则时，下发的配置项格式与 [API 文档](https://www.envoyproxy.io/docs/envoy/latest/api/api) 中给出的相同。以 listener 的下发为例，下发的 listener 与配置文件的 static_resources 中的 listeners 的格式相同。[Listener 的格式](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#listener) 如下：

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

go-control-plane 中对应的定义在文件 [envoy/api/v2/lds.pb.go](https://github.com/envoyproxy/go-control-plane/blob/v0.8.4/envoy/api/v2/lds.pb.go) 中：

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

[1]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto#envoy-api-msg-config-filter-network-http-connection-manager-v2-httpconnectionmanager  "HTTP Connection Manager"
[2]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/network "network filter"
[3]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/route/route.proto#route-virtualhost  "route-virtualhost"
[4]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/rds.proto#envoy-api-msg-routeconfiguration "routeconfiguration"
[5]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/route/route.proto "route.proto"
[6]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cds.proto#cluster "cluster"
[7]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/endpoint/endpoint.proto  "endpoint.proto"
[8]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#listener "listener"
[9]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/auth/cert.proto#envoy-api-msg-auth-secret "secret"
[10]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto#config-bootstrap-v2-bootstrap-dynamicresources)、[lds](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/bootstrap/v2/bootstrap.proto#config-bootstrap-v2-bootstrap-dynamicresources)、[rds](https://www.envoyproxy.io/docs/envoy/latest/configuration/http_conn_man/rds.html "CDS/LDS/ADS"
[11]: https://www.envoyproxy.io/docs/envoy/latest/configuration/secret.html  "SDS"

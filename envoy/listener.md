<!-- toc -->
# Envoy 的 Listener 配置详解

[Listener][1] 是 envoy 最重要的配置，也是最复杂的配置，它是 envoy 的监听配置，指定了 envoy 监听地址，以及请求如何处理、转发到哪里。Listener 中可以包含多个不同 filter，有一些 filter 本身又是比较复杂的，譬如 [HTTP Connection Manager][2]。

## Listener 配置格式

Listener 的配置格式如下，可以在 [api 文档][2] 中找到：

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

其中 [address][3] 是监听地址，[filter_chains][4] 和 [listener_filters][5] 是 Listener 的配置中最重要的也最复杂的，剩余的都是一些细节配置，相对简单一些。


[1]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/listeners/listeners "Listeners"
[2]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto#envoy-api-msg-config-filter-network-http-connection-manager-v2-httpconnectionmanager  "HTTP Connection Manager"
[3]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/address.proto#envoy-api-msg-core-address "core.Address"
[4]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-msg-listener-filterchain "listener.FilterChain"
[5]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-msg-listener-listenerfilter  "listener.ListenerFilter"
[6]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/config#  "Extensions"
[7]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/29/envoy-07-features-2-dynamic-discovery.html#go-control-plane "go-control-plane"
[8]: https://github.com/envoyproxy/go-control-plane/blob/v0.8.4/envoy/api/v2/lds.pb.go "go-control-plane/envoy/api/v2/lds.pb.go"
[9]: https://github.com/envoyproxy/go-control-plane/blob/v0.8.4/envoy/api/v2/listener/listener.pb.go "envoy/api/v2/listener/listener.pb.go"
[10]: https://github.com/envoyproxy/go-control-plane/tree/v0.8.4/envoy/config/filter "go-control-plane/envoy/config/filter/"

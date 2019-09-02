<!-- toc -->
# Envoy 的 Listener 中的 Filter 详解

从 socket 中收取的请求先经过 [listener_filters][5] 处理，然后再由 [filter_chains][4] 处理，前者包含的 filter 称为 listener filter，后者包含的 filter 称为 network filter。因为 listener_filters 先起作用，因此它可以修改请求的信息，从而影响 filter_chains 的匹配。

[filter][11] 的文档在 [Extensions][12] 中，当前（2019-08-05 20:35:10）有以下几个大类：

```sh
Listener filters
Network filters
HTTP filters
Thrift filters
Common access log types
Common fault injection types
Dubbo filters
```

[Configuration reference][13] 中有一半的内容在介绍 filter，对每个 filter 的用途和用法作了简要介绍。

## 参考

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
[11]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/filter "Filters"
[12]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/config#  "Extensions"
[13]: https://www.envoyproxy.io/docs/envoy/latest/configuration/configuration  "Configuration reference"

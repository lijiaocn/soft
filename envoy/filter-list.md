#  Envoy 的 Listener 中可以使用的 filter

都有哪些 filters 可以使用？

[filter][2] 的文档在 [Extensions][1] 中，当前（2019-08-05 20:35:10）有以下几个大类：

```sh
Network filters
HTTP filters
Thrift filters
Common access log types
Common fault injection types
Listener filters
Dubbo filters
```

在 [filter][2] 中可以看到每个大类下的清单：

[![envoy filters 列表网页预览](/img/envoy/envoy-filters.png)][2]

让人搞不清楚的是：哪些 filter 可用作为 listener_filters 使用，哪些可以作为 filter_chains 中的 filter 使用？

现在可以确定 [HTTP filters][3] 是在 [HTTP connection manager][4] 中使用的，HTTP connection manager 是 [Network filters][5] 中的一员。

[1]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/config#  "Extensions"
[2]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/filter "Filters"
[3]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/http "HTTP filters"
[4]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto "HTTP connection manager"
[5]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/network "Network filters"

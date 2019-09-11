<!-- toc -->
# Envoy 的 Listener 配置详解

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

[Listener][1] 是 envoy 最重要的配置，也是最复杂的配置，它是 envoy 的监听配置，指定了 envoy 监听地址，以及请求如何处理、转发到哪里。Listener 中可以包含多个不同 filter，有一些 filter 本身又是比较复杂的，可以继续包含 filter，譬如 [HTTP Connection Manager](./network-filter.md#envoyhttpconnectionmanager)。

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

其中 [address][3] 是监听地址，[filter_chains][4] 和 [listener_filters][5] 是 listener 中最重要也最复杂的配置，剩余的都是一些细节配置，相对简单一些。

**注意事项：**

* listener 的监听地址是互斥的，两个 listener 不能监听同一个 socket 地址
* listener_filters 是 [listener filter](./listener-filter.md)
* filter_chains 是 [network filter](./network-filter.md)，可以有多组

go-control-plane 中的 listener 在 [envoy/api/ve/lds.pb.go][11] 中定义，每个字段都有非常详细的注释，[api文档][2] 就是通过这些注释生成的。

## 参考

[1]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/listeners/listeners "Listeners"
[2]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#envoy-api-msg-listener "Listener configuration overview"
[3]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/address.proto#envoy-api-msg-core-address "core.Address"
[4]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-msg-listener-filterchain "listener.FilterChain"
[5]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-msg-listener-listenerfilter  "listener.ListenerFilter"
[6]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/config#  "Extensions"
[7]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/29/envoy-07-features-2-dynamic-discovery.html#go-control-plane "go-control-plane"
[8]: https://github.com/envoyproxy/go-control-plane/blob/v0.8.4/envoy/api/v2/lds.pb.go "go-control-plane/envoy/api/v2/lds.pb.go"
[9]: https://github.com/envoyproxy/go-control-plane/blob/v0.8.4/envoy/api/v2/listener/listener.pb.go "envoy/api/v2/listener/listener.pb.go"
[10]: https://github.com/envoyproxy/go-control-plane/tree/v0.8.4/envoy/config/filter "go-control-plane/envoy/config/filter/"
[11]: https://github.com/envoyproxy/go-control-plane/blob/master/envoy/api/v2/lds.pb.go#L67 "type Listener struct"

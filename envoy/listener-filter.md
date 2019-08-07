<!-- toc -->
# Envoy 的 listener filter 列表

下面是可以在加入到 listener_filter 的 listener filter。 这些 filter 的主要作用是检测协议、解析协议，通过它们解析出的信息用于匹配 filter_chains 中的 filter。

## name: envoy.listener.http_inspector

[HTTP Inspector][7] 判断应用层的数据是否使用 HTTP 协议，如果是，续继判断 HTTP 协议的版本号（HTTP 1.0、HTTP 1.1、HTTP 2），: 

```yaml
listener_filters:
  - name: "envoy.listener.http_inspector"
    typed_config: {}
```

## name: envoy.listener.original_dst

[Original Destination][8] 用来读取 socket 的配置项 `SO_ORIGINAL_DST`，在使用 [透明代理模式][9] 时用到。透明代理模式下，代理进程可以获取报文的原始目地址，在 envoy 中，用该 filter 获取。[Linux系统知识/透明代理](../linuxsys/tpproxy.md) 对透明代理有详细介绍。

```yaml
listener_filters:
  - name: "envoy.listener.original_dst"
```

## name: envoy.listener.original_src

[Original Source][10] 类似于 lvs 的 [DR 模式][11] ，假设 downstream 的 IP 是 10.1.2.3，envoy 将报文转发给 upstream 时，使用同样的源 IP，upstream 看到的源 IP 是原始的 IP，不是 envoy 的 IP。

与 lvs 的 [DR 模式][11] 不同的是，在 lvs 中，upsteram 是直接将回应包发送给 downstream，而 envoy 的文档中强调，必须通过配置网络环境，让 uptream 的回应包发送到 envoy ，由 envoy 转发。

由此判断， [Original Source][10]  的用途是让 uptream 看到的是请求端的 IP，让双方感知不到 envoy 的存在。

下面是一个使用示例，用到了两个 filter: envoy.listener.proxy_protocol 的用途是从代理协议中解析出真实的源 IP，不是必须的，该 filter 的用法见下一节；envoy.listener.original_src 透传原始的源 IP。

```yaml
listeners:
- address:
    socket_address:
      address: 0.0.0.0
      port_value: 8888
  listener_filters:
    - name: envoy.listener.proxy_protocol
    - name: envoy.listener.original_src
      config:
        mark: 123
```

`mark 123` 是为被透传的报文打上 mark 123，当 upstream 和 envoy 位于同一台机器上时，将标记了该 mark 的报文转发到本地: 

```sh
iptables  -t mangle -I PREROUTING -m mark     --mark 123 -j CONNMARK --save-mark
iptables  -t mangle -I OUTPUT     -m connmark --mark 123 -j CONNMARK --restore-mark
ip6tables -t mangle -I PREROUTING -m mark     --mark 123 -j CONNMARK --save-mark
ip6tables -t mangle -I OUTPUT     -m connmark --mark 123 -j CONNMARK --restore-mark
ip rule add fwmark 123 lookup 100
ip route add local 0.0.0.0/0 dev lo table 100
ip -6 rule add fwmark 123 lookup 100
ip -6 route add local ::/0 dev lo table 100
echo 1 > /proc/sys/net/ipv4/conf/eth0/route_localnet
```

上面的设置规则用到一个 linux 知识: [local 地址的认定](../linuxsys/localip.md)。

如果 envoy 和 upstream  不在同一个 host，需要在网络上设置。

## name: envoy.listener.proxy_protocol

[Proxy Protocol][12] 现在（2019-08-06 17:33:53）能够解析代理协议，如果请求是代理转发过来的，用该 filter 可以解析出真实的源 IP。

具体支持的代理协议未知，已知支持 [HAProxy Proxy Protocol][13]。

```yaml
listener_filters:
  - name: envoy.listener.proxy_protocol
```

## name: envoy.listener.tls_inspector

[TLS Inspector][14] 用来判断是否启用了 TLS 加密，如果是 TLS 协议，解析出 Server Name、Negotiation 信息。解析出来的信息用于 FilterChain 的匹配。

```yaml
listener_filters:
  - name: "envoy.listener.tls_inspector"
    typed_config: {}
```

## 参考

[1]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/config#  "Extensions"
[2]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/filter "Filters"
[3]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/http/http "HTTP filters"
[4]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto "HTTP connection manager"
[5]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/network "Network filters"
[6]: https://www.envoyproxy.io/docs/envoy/latest/configuration/configuration  "Configuration reference"
[7]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listener_filters/http_inspector "HTTP Inspector"
[8]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listener_filters/original_dst_filter "Original Destination"
[9]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/lds.proto#envoy-api-field-listener-transparent  "transparent"
[10]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listener_filters/original_src_filter "Original Source"
[11]: http://www.linuxvirtualserver.org/VS-DRouting.html "Virtual Server via Direct Routing"
[12]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listener_filters/proxy_protocol#config-listener-filters-proxy-protocol "Proxy Protocol"
[13]: https://www.haproxy.org/download/1.9/doc/proxy-protocol.txt "The PROXY protocol"
[14]: https://www.envoyproxy.io/docs/envoy/latest/configuration/listener_filters/tls_inspector "TLS Inspector"

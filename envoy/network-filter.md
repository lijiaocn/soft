<!-- toc -->

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

# Envoy 的 network filter 列表


可以加入到 filter_chains 中的 network filter 数量众多，[Network filters][1] 中列出了以下几个：

	Dubbo proxy
	Client TLS authentication
	Echo
	External Authorization
	Mongo proxy
	MySQL proxy
	Rate limit
	Role Based Access Control (RBAC) Network Filter
	Redis proxy
	TCP proxy
	Thrift proxy
	Upstream Cluster from SNI
	ZooKeeper proxy

另还有一个名为 HTTP connection manager 的 network filter，这个 filter 主要处理 http 协议，自身已经足够复杂，被单独列出 [HTTP connection manager][2]。

HTTP connection manager、Thrift proxy 和 Dubbo proxy 还有在自己内部使用的 filter，分别是： [HTTP filters][3]、[Thrift filters][4]、[Dubbo filters][5]。

## envoy.tcp_proxy

[TCP proxy][12] 管理 download client 与 upstream cluster 之间的 tcp 连接，保证连接数不超过 upstream cluster 的上限。通常和其它 filter 配合使用。

## envoy.ratelimit

[Rate limit][13] 提供全局限速功能（需要连接外部的限速服务），可以限制 tcp 连接速率和 http 请求速率。为了避免每个连接、或者每个请求都查询限速服务，可以设置限速服务的查询占比：

```sh
ratelimit.tcp_filter_enabled     # 对应比例的连接会查询限速服务，但不执行查询结果
ratelimit.tcp_filter_enforcing   # 对应比例的连接会查询限速服务，并按照查询结果执行
```

## envoy.filters.network.rbac

[Role Based Access Control (RBAC) Network Filter][14] 提供了访问控制的能力。

## envoy.client_ssl_auth

[Client TLS authentication][7] 验证 client 端的证书，它会以配置的频率调用 ` GET /v1/certs/list/approved` 获取最新的有效证书。

## envoy.ext_authz

[External Authorization][9] 通过外部的认证服务判断当前请求是否获得授权，如果没有授权，关闭连接。

```yaml
filters:
  - name: envoy.ext_authz
    config:
      stat_prefix: ext_authz
      grpc_service:
        envoy_grpc:
          cluster_name: ext-authz

clusters:
  - name: ext-authz
    type: static
    http2_protocol_options: {}
    load_assignment:
      cluster_name: ext-authz
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 10003
```

## envoy.echo

[Echo][8] 将收到的数据原样返回给客户端。

## envoy.http_connection_manager

[HTTP connection manager][18] 是专门处理 http 协议的 network filter，因为 http 最经常使用的协议，对代理功能需求也非常多样， HTTP connection manager 本身是一个比较复杂的 network filter，在 envoy 文档中被单独列出：[HTTP connection manager][18]。

HTTP connection manager 有自己专用的 HTTP filters，在 [http filters](./http-filter.md)  中单独介绍。

## envoy.filters.network.thrift_proxy

[Thrift proxy][16] 能够解析 thrift 协议。

Thrift proxy 有自己专用的 [Thrift filters][4]。

## envoy.filters.network.dubbo_proxy

[Dubbo proxy][6] 解析 dubbo client 和 service 之间的 grpc 通信。

Dubbo proxy 有自己专用的 [Dubbo filters][5]。

```yaml
filter_chains:
- filters:
  - name: envoy.filters.network.dubbo_proxy
    config:
      stat_prefix: dubbo_incomming_stats
      protocol_type: Dubbo
      serialization_type: Hessian2
      route_config:
        name: local_route
        interface: org.apache.dubbo.demo.DemoService
        routes:
        - match:
            method:
              name:
                exact: sayHello
          route:
            cluster: user_service_dubbo_server
      dubbo_filters:
      - name: envoy.filters.dubbo.testFilter
        config:
          "@type": type.googleapis.com/google.protobuf.Struct
          value:
            name: test_service
      - name: envoy.filters.dubbo.router
```

## envoy.filters.network.zookeeper_proxy

[ZooKeeper proxy][17]  能够解析 zookeeper 协议。


## envoy.mongo_proxy

[Mongo proxy][10] 能够解析 mongo 通信，记录 mongo 日志、统计、注入错误等。

## envoy.filters.network.mysql_proxy

[MySQL proxy][11] 能够解析 mysql 的通信协议，需要和 [TCP proxy][] 一起使用：

```yaml
filter_chains:
- filters:
  - name: envoy.filters.network.mysql_proxy
    typed_config:
      "@type": type.googleapis.com/envoy.config.filter.network.mysql_proxy.v1alpha1.MySQLProxy
      stat_prefix: mysql
  - name: envoy.tcp_proxy
    typed_config:
      "@type": type.googleapis.com/envoy.config.filter.network.tcp_proxy.v2.TcpProxy
      stat_prefix: tcp
      cluster: ...
```

## envoy.redis_proxy

[Redis proxy][15] 能够解析 redis 协议，使 envoy 成为 redis 代理，可以将不同的 redis command 代理到不同 redis cluster。

## 参考

[1]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/network_filters "Network filters"
[2]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/http_conn_man/http_conn_man "HTTP connection manager"
[3]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/http_filters/http_filters "HTTP filters"
[4]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/thrift_filters/thrift_filters "Thrift filters"
[5]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/dubbo_filters/dubbo_filters "Dubbo filters"
[6]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/dubbo_proxy_filter "Dubbo proxy"
[7]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/client_ssl_auth_filter "Client TLS authentication"
[8]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/echo_filter "Echo"
[9]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/ext_authz_filter "External Authorization"
[10]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/mongo_proxy_filter "Mongo proxy"
[11]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/mysql_proxy_filter "MySQL proxy"
[12]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/tcp_proxy_filter "TCP proxy"
[13]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/rate_limit_filter "Rate limit"
[14]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/rbac_filter "Role Based Access Control (RBAC) Network Filter"
[15]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/redis_proxy_filter "Redis proxy"
[16]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/thrift_proxy_filter "Thrift proxy"
[17]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/network_filters/zookeeper_proxy_filter "ZooKeeper proxy"
[18]: https://www.envoyproxy.io/docs/envoy/v1.11.0/configuration/http_conn_man/http_conn_man "HTTP connection manager"

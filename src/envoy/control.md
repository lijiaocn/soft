<!-- toc -->

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

# Envoy 的控制平面的实现


可以动态下发的配置有：

* [cluster][6]
* [cluster][6] 中的 [endpoint][7]
* [listener][8]
* [listener][8] 的 [Network filter][2] 中的 [HTTP connection manager][1] 中的 [Virtualhost][3] 中的 [RouteConfiguration][4] 中的 [route][5]
* [serect][9]

对应的服务端分别称为 [cds][10]、[lds][13]、[rds][14]、[sds][11] 。

示例代码：

```sh
git clone https://github.com/introclass/go-code-example.git
```

## 下发协议

[envoyproxy/data-plane-api][15] 定义了 envoy 的 REST API 和配置下发时使用的通信协议 [xDS REST and gRPC protocol][16]，同时 Envoy 开源了一个控制层面的开发框架 [envoyproxy/go-control-plane][17]，使用该框架可以快速开发一个控制平面（也就是 xDS 服务），框架已经实现了通信过程，不需要自行实现通信协议。

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

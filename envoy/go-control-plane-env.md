<!-- toc -->
# go-control-plane 下发配置示例—环境准备

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

这里演示一个用 go-control-plane 下发配置的例子。

演示用的代码和配置文件位于：[github.com/introclass/go-code-example/envoydev/xds][1]。xds.go 是用 go-controle-plane 实现的简陋控制平面，envoy-docker-run 目录中是启动 envoy 容器的文件：

```sh
├── envoy-docker-run
│   ├── envoy-0-default.yaml
│   ├── envoy-1-ads-with-xds.yaml
│   ├── envoy-1-ads.yaml
│   ├── envoy-1-static.yaml
│   ├── envoy-1-xds.yaml
│   ├── envoy-to-grpc-svc.yaml
│   ├── log
│   │   └── admin_access.log
│   └── run.sh
└── xds.go
```

## 启动 envoy

envoy 启动时使用用配置文件 [envoy-1-ads-with-xds.yaml][2]，这个配置文件中

```sh
./run.sh envoy-1-ads-with-xds.yaml
```

[envoy-1-ads-with-xds.yaml][2] 中配置了两个 cluster，一个是 ads_cluster 一个是 xds_cluster，后面的演示代码既会下发使用 xds ，也会使用 ads ，所以在配置了这两个 cluster。


```sh
│   ├── envoy-0-default.yaml         # envoy 容器中的默认配置
│   ├── envoy-0-example.yaml         # 初次体验使用的配置
│   ├── envoy-1-ads-with-xds.yaml    # 演示配置下发时用的配置，同时配置了 ads、xds
│   ├── envoy-1-ads.yaml             # 只使用 ads 发现配置的配置 
│   ├── envoy-1-static.yaml          # 完全静态的配置
│   ├── envoy-1-xds.yaml             # 只使用 ads 发现配置的配置
│   ├── envoy-to-grpc-svc.yaml       # grpc 代理配置
```

## 演示用的配置文件说明

[envoy-1-ads-with-xds.yaml][2] 的内容比较长，分成几段说明。

第一段是 envoy 的常规配置，配置 envoy 的 id、管理接口等：


```yaml
node:
  id: "envoy-64.58"
  cluster: "test"
#runtime:
#  symlink_root: /srv/runtime/current
#  subdirectory: envoy
#  override_subdirectory: envoy_override
watchdog:
  miss_timeout: 0.2s
  megamiss_timeout: 1s
  kill_timeout: 0s
  multikill_timeout: 0s
flags_path: /etc/envoy/flags/
stats_flush_interval: 5s
stats_config:
  use_all_default_tags: true
stats_sinks:
  name: envoy.stat_sinks.hystrix
  config:
    num_buckets: 10
admin:
  access_log_path: /var/log/envoy/admin_access.log
  profile_path: /var/log/envoy/envoy.prof
  address:
    socket_address:
      protocol: TCP
      address: 0.0.0.0
      port_value: 9901
```

第二段是发现配置，这里采用的是 ads 的方式，cds 和 lds 都指向 ads：

```yaml
dynamic_resources:
  ads_config:
    api_type: GRPC
    grpc_services:
      envoy_grpc:
        cluster_name: ads_cluster
  cds_config: {ads: {}}
  lds_config: {ads: {}}
```

第三段是下发配置，两个 cluster，一个是 ads_cluster，在上一段配置中已经被引用，另一个 xds_cluster在配置文件中没有被用到，但是后面的演示代码下发的配置中会用到。

ads_cluster 和 xds_cluster 的地址都是 127.0.0.1:5678，演示环境中 xds.go 和 envoy 容器在一台机器上，所以这里配置的都是本地地址。

```yaml
static_resources:
  clusters:
  - name: ads_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    load_assignment:
      cluster_name: ads_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 5678
  - name: xds_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    load_assignment:
      cluster_name: xds_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 5678
```

最后 static_resources 中还有两个演示中没有用到的 cluster，一个是指向 8.8.8.8:53 ，一个指向  www.baidu.com，它们只是用来示范 cluster 的配置方法：

```yaml
  - name: dns_google
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    load_assignment:
      cluster_name: dns_google
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 8.8.8.8
                port_value: 53
  - name: service_baidu
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    # Comment out the following line to test on v6 networks
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_baidu
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: www.baidu.com
                port_value: 443
    tls_context:
      sni: www.baidu.com
```

## 参考

[1]: https://github.com/introclass/go-code-example/tree/master/envoydev/xds  "github.com/introclass/go-code-example/envoydev/xds"
[2]: https://github.com/introclass/go-code-example/blob/master/envoydev/xds/envoy-docker-run/envoy-1-ads-with-xds.yaml "envoy-1-ads-with-xds.yaml"

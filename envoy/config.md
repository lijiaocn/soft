<!-- toc -->
# Envoy 的配置文件格式

Envoy 的 [API 文档](https://www.envoyproxy.io/docs/envoy/latest/api/api) 中，分别给出了每个配置项的格式，[《Envoy Proxy使用介绍教程（五）：envoy的配置文件完全展开介绍》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/27/envoy-05-configfile.html) 将 envoy 1.8.0  的各个配置项格式组合了起来，呈现了 envoy 配置文件的完整轮廓，如下：

```json
{
  "node": {
    "id": "...",
    "cluster": "...",
    "metadata": "{...}",
    "locality": "{...}",
    "build_version": "..."
  },
  "static_resources": {
    "listeners": [],
    "clusters": [],
    "secrets": []
  },
  "dynamic_resources": {
    "lds_config": "{...}",
    "cds_config": "{...}",
    "ads_config": "{...}"
  },
  "cluster_manager": {
    "local_cluster_name": "...",
    "outlier_detection": "{...}",
    "upstream_bind_config": "{...}",
    "load_stats_config": "{...}"
  },
  "hds_config": {
    "api_type": "...",
    "cluster_names": [],
    "grpc_services": [],
    "refresh_delay": "{...}",
    "request_timeout": "{...}",
    "rate_limit_settings": "{...}"
  },
  "flags_path": "...",
  "stats_sinks": [
    {
      "name": "...",
      "config": "{...}"
    }
  ],
  "stats_config": {
    "stats_tags": [],
    "use_all_default_tags": "{...}",
    "stats_matcher": "{...}"
  },
  "stats_flush_interval": "{...}",
  "watchdog": {
    "miss_timeout": "{...}",
    "megamiss_timeout": "{...}",
    "kill_timeout": "{...}",
    "multikill_timeout": "{...}"
  },
  "tracing": {
    "http": "{...}"
  },
  "rate_limit_service": {
    "grpc_service": "{...}"
  },
  "runtime": {
    "symlink_root": "...",
    "subdirectory": "...",
    "override_subdirectory": "..."
  },
  "admin": {
    "access_log_path": "...",
    "profile_path": "...",
    "address": "{...}"
  },
  "overload_manager": {
    "refresh_interval": "{...}",
    "resource_monitors": [],
    "actions": []
  }
}
```

## 本手册中用到的几个配置文件

[xds/envoy-docker-run][1]：

```sh
│   ├── envoy-0-default.yaml         # envoy 容器中的默认配置
│   ├── envoy-0-example.yaml         # 初次体验使用的配置
│   ├── envoy-1-ads-with-xds.yaml    # 演示配置下发时用的配置，同时配置了 ads、xds
│   ├── envoy-1-ads.yaml             # 只使用 ads 发现配置的配置 
│   ├── envoy-1-static.yaml          # 完全静态的配置
│   ├── envoy-1-xds.yaml             # 只使用 ads 发现配置的配置
│   ├── envoy-to-grpc-svc.yaml       # grpc 代理配置
```

envoy-0-default.yaml 是 envoy 容器中的默认配置文件，内容如下：

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      protocol: TCP
      address: 127.0.0.1
      port_value: 9901
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  host_rewrite: www.google.com
                  cluster: service_google
          http_filters:
          - name: envoy.router
  clusters:
  - name: service_google
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    # Comment out the following line to test on v6 networks
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_google
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: www.google.com
                port_value: 443
    tls_context:
      sni: www.google.com
```


## 参考

[1]: https://github.com/introclass/go-code-example/tree/master/envoydev/xds/envoy-docker-run "envoy-docker-run"

# Envoy 的静态配置示例

监听转发配置（listener、cluster）可以在 `static_resources` 中静态配置。

下面的配置文件中，静态配置了两个 listener 分别监听端口 9000 和 10000，将请求分别转发给静态配置的两个 cluster，一个 cluster 配置的是 IP，一个配置的是域名：

```yaml
node:
  id: "envoy-64.58"
  cluster: "test"
#runtime:
#  symlink_root: /srv/runtime/current
#  subdirectory: envoy
#  override_subdirectory: envoy_override
#flags_path: /etc/envoy/flags/
watchdog:
  miss_timeout: 0.2s
  megamiss_timeout: 1s
  kill_timeout: 0s
  multikill_timeout: 0s
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
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 9000
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          stat_prefix: ingress_http
          generate_request_id: true
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["webshell.com"]
              routes:
              - match:
                  prefix: "/"
                route:
                  host_rewrite: webshell.com
                  cluster: service_webshell
          http_filters:
          - name: envoy.router
            config:
              dynamic_stats: false
  - name: listener_1
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
                  host_rewrite: www.baidu.com
                  cluster: service_baidu
          http_filters:
          - name: envoy.router
  clusters:
  - name: service_webshell
    connect_timeout: 0.25s
    type: static
    lb_policy: ROUND_ROBIN
    hosts:
      - socket_address:
          address:  172.16.128.171
          port_value: 8080
  - name: service_baidu
    connect_timeout: 0.25s
    type: LOGICAL_DNS
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

访问 127.0.0.1:9000 时，host 为 webshell.com 时，转发都IP 172.16.128.171，这是一个 echo 会显服务，可以看到回显的请求：

```sh
$ curl  -H "Host: webshell.com"  127.0.0.1:9000

Hostname: echo-7df87d5c6d-s4vhq

Pod Information:
   -no pod information available-

Server values:
   server_version=nginx: 1.13.3 - lua: 10008

Request Information:
   client_address=10.10.64.58
   method=GET
   real path=/
   query=
   request_version=1.1
   request_uri=http://webshell.com:8080/

Request Headers:
   accept=*/*
   content-length=0
   host=webshell.com
   user-agent=curl/7.29.0
   x-envoy-expected-rq-timeout-ms=15000
   x-forwarded-proto=http
   x-request-id=ee6d6444-4054-46bb-9ae2-9924b6bcaa14

Request Body:
   -no body in request-
```

访问 127.0.0.1:10000 时，无论 host 是多少，都转发到 www.baidu.com：

```sh
$ curl 127.0.0.1:10000
...
<title>百度一下，你就知道</title>
...
```

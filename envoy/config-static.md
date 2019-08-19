<!-- toc -->
# Envoy 的静态配置示例

监听转发配置（listener、cluster）可以静态配置也可以动态获取，在 `static_resources` 中静态配置。

这里使用的配置文件是：[envoy-1-static.yaml][1]，envoy 启动命令为：

```sh
./run.sh envoy-1-static.yaml
```

## 转发到指定IP

在 clusters 中配置了一个名为 service_echo 的 cluster ，它指向 172.17.0.3:8080，在本例中，这是前面启动的 echo 容器的地址：

```yaml
- name: service_echo
  connect_timeout: 0.25s
  type: static
  lb_policy: ROUND_ROBIN
  hosts:
    - socket_address:
        address:  172.17.0.3
        port_value: 8080
```

在 listeners 中配置一个名为 listener_0  的 listener，它监听 80 端口，将匹配 "Host: echo.com" 和 "/" 的请求转发到上面的 cluster：

```yaml
- name: listener_0
  address:
    socket_address:
      protocol: TCP
      address: 0.0.0.0
      port_value: 80
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
            domains: ["echo.com"]
            routes:
            - match:
                prefix: "/"
              route:
                host_rewrite: echo.com
                cluster: service_echo
        http_filters:
        - name: envoy.router
          config:
            dynamic_stats: false
```

直接访问 listener_0，不带 host 时，返回 404：

```sh
$ curl -v  127.0.0.1:80
* Rebuilt URL to: 127.0.0.1:80/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 404 Not Found
< date: Fri, 16 Aug 2019 11:50:13 GMT
< server: envoy
< content-length: 0
<
* Connection #0 to host 127.0.0.1 left intact
```

带上 host，返回 echo 的响应：

```sh
$ curl 127.0.0.1:80 -H "Host: echo.com"

Hostname: 611185215d7a

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.2
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://echo.com:8080/

Request Headers:
	accept=*/*
	content-length=0
	host=echo.com
	user-agent=curl/7.54.0
	x-envoy-expected-rq-timeout-ms=15000
	x-forwarded-proto=http
	x-request-id=02a1f65e-7ae0-471a-acfc-913633ff54f3

Request Body:
	-no body in request-
```

## 转发到域名

配置一个 cluster，endpoint 中填入的是域名：

```yaml
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

配置一个 listener 监听 81 端口，路由规则为将所有请求转发到 www.baidu.com，转发时改写 host：

```yaml
- name: listener_1
  address:
    socket_address:
      protocol: TCP
      address: 0.0.0.0
      port_value: 81
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
```

访问 127.0.0.1:81 时，无论 host 是多少，都转发到 www.baidu.com：

```sh
$ curl 127.0.0.1:81
...
<title>百度一下，你就知道</title>
...
```

## 参考

[1]: https://github.com/introclass/go-code-example/blob/master/envoydev/xds/envoy-docker-run/envoy-1-static.yaml "envoy-1-static.yaml"
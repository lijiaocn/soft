<!-- toc -->
# 用 echoserver 观察代理/转发效果

echoserver 是一个回显请求的 http 服务，用来观察 http 请求的代理/转发效果非常方便。

## 启动 echoserver 容器

准备一个 echoserver 观察 envoy 转发来的请求。

下载镜像：

```sh
docker pull googlecontainer/echoserver:1.10
```

启动：

```sh
$ docker run -idt  -p 8080:8080 -p 8443:8443 googlecontainer/echoserver:1.10
```

直接访问 echo 容器效果如下：

```sh
$ curl 127.0.0.1:8080
Hostname: 611185215d7a

Pod Information:
    -no pod information available-

Server values:
    server_version=nginx: 1.13.3 - lua: 10008

Request Information:
    client_address=172.17.0.1
    method=GET
    real path=/
    query=
    request_version=1.1
    request_scheme=http
    request_uri=http://127.0.0.1:8080/

Request Headers:
    accept=*/*
    host=127.0.0.1:8080
    user-agent=curl/7.54.0

Request Body:
    -no body in request-
```

## envoy 代理 echoserver

先获取 echoserver 容器的 IP 地址，确定 envoy 能够访问这个地址：

```sh
$ docker inspect 611185215d7a -f "{{.NetworkSettings.Networks.bridge.IPAddress}}"
172.17.0.3
```

## 配置 cluster

在 static_resources: -> clusters: 中添加下面的配置，这是 echoserver 容器的地址：

```yaml
- name: service_echo
  connect_timeout: 0.25s
  type: STATIC
  lb_policy: ROUND_ROBIN
  http2_protocol_options: {}
  load_assignment:
    cluster_name: service_echo
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address:  172.17.0.3
              port_value: 8080
```

## 配置 listener

在 static_resources: -> listeners: 中添加下面配置，这个 listener 监听 80 端口，将 host 匹配 "*" 和 prefix 匹配 "/" 的请求转发给上面配置的 cluster，并且在转发的时候将 host 修改为 www.google.com：

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
                cluster: service_echo
        http_filters:
        - name: envoy.router
```

## 启动，观察效果

上面的配置已经添加到 envoy-0-example.yaml 中，直接用下面的命令启动：

## 参考

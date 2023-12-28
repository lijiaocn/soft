<!-- toc -->

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

# 用 echoserver 观察代理/转发效果

echoserver 是一个回显用户请求的 http 服务，用来观察 http 请求的代理/转发效果非常方便。

echoserver 1.10 不支持 HTTP 2.0。

## 启动 echoserver 容器

准备一个 echoserver 观察 envoy 转发来的请求，echoserver 用途见 [HTTP 协议相关的工具](../tools/http.md)。

```sh
$ docker run -idt --name echoserver -p 9090:8080 -p 9443:8443 googlecontainer/echoserver:1.10
```

## 获取 echoserver 容器的 IP

先获取 echoserver 容器的 IP 地址，确定 envoy 能够访问这个地址：

```sh
# echoserver 是容器的名字，替换成你自己的容器名
$ docker inspect echoserver -f "{{.NetworkSettings.Networks.bridge.IPAddress}}"
172.17.0.2
```

## 配置 cluster

该示例使用的配置文件是 [envoy-0-example.yaml][1]，在 static_resources: -> clusters: 中配置的是 echoserver 容器的地址，注意将 address 替换成你自己的 echoserver 容器的 IP：

```yaml
- name: service_echo
  connect_timeout: 0.25s
  type: STATIC
  lb_policy: ROUND_ROBIN
  #http2_protocol_options: {}  # 注意 echoserver 不支持http 2.0，不能有这项配置
  load_assignment:
    cluster_name: service_echo
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address:  172.17.0.2
              port_value: 8080
```

## 配置 listener

该示例使用的配置文件是 [envoy-0-example.yaml][1]，在 static_resources: -> listeners: 中配置的是 listener，这个 listener 监听 80 端口，将 host 匹配 "*" 和 prefix 匹配 "/" 的请求转发给上面配置的 cluster，并且在转发的时候将 host 修改为 www.google.com：

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
                host_rewrite: www.baidu.com
                cluster: service_echo
        http_filters:
        - name: envoy.router
```

## 启动，观察效果

上面的配置已经添加到 [envoy-0-example.yaml][1] 中，直接用下面的命令启动：

```sh
$ ./run.sh envoy-0-example.yaml
```

访问 envoy 的 80 端口，效果如下，注意观察 echoserver 收到的请求的 host 是 www.baidu.com，这是因为上面的配置里有一项 “host_rewrite: www.baidu.com”：

```
$ curl 127.0.0.1:80

Hostname: 7759cabd7402

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.3
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://www.baidu.com:8080/

Request Headers:
	accept=*/*
	content-length=0
	host=www.baidu.com     # echoserver 收到的请求的 host
	user-agent=curl/7.54.0
	x-envoy-expected-rq-timeout-ms=15000
	x-forwarded-proto=http
	x-request-id=957e0bd8-2fb1-4ff1-8131-f1fff1cb0e9a

Request Body:
	-no body in request-
```

## 参考

[1]: https://github.com/introclass/go-code-example/blob/master/envoydev/xds/envoy-docker-run/envoy-0-example.yaml "envoy-0-example.yaml"

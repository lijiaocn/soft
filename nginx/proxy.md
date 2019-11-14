<!-- toc -->
# 使用 nginx 实现本地透明代理

尝试用 nginx 作为透明代理，修改报文头。

## 准备目标服务

目标服务使用 [试验环境](./env.md) 中启动的 echoserver，监听地址为 127.0.0.1:9090。

为目标服务准备一个域名 echo.example，用本地的 dnsmasq 解析。

下面是在 mac 上的操作。

* 部署 dnsmasq：

```sh
$ sudo chown -R $(whoami):admin /usr/local
$ brew install dnsmasq
```

* 在 /usr/local/etc/dnsmasq.conf 中添加解析，解析到本地地址 127.0.0.1：

```conf
port 8053     # 注意，不要用 53 端口，否则需要用 root 运行
address=/echo.example/127.0.0.1
```

dnsmasq 监听端口最好不用 53 ，mac 的权限要求 53 端口必须用 root 身份监听，见 [Restart dnsmasq without sudo][2]。

* 启动 dnsmasq：

```sh
$ brew services start dnsmasq
```

验证解析：

```sh
$ dig @127.0.0.1 -p 8053 echo.example

; <<>> DiG 9.10.6 <<>> @127.0.0.1 -p 8053 echo.example
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49022
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;echo.example.            IN    A

;; ANSWER SECTION:
echo.example.        0    IN    A    127.0.0.1

;; Query time: 41 msec
;; SERVER: 127.0.0.1#8053(127.0.0.1)
;; WHEN: Wed Oct 30 17:15:03 CST 2019
;; MSG SIZE  rcvd: 57
```

## 配置 nginx

配置nginx：

```conf
server {
    listen       9000;
    listen       [::]:9000;
    keepalive_requests  2000;
    keepalive_timeout 60s;

    location / {
        resolver 127.0.0.1:8053;  # 必须配置域名服务器
        # 这里因为 nginx 和 目标服务在同一台机器上，使用了不同端口，所以写成 $host:9090
        # 如果位于不同机器上，写成：proxy_pass  http://$host:9000$request_uri;
        proxy_pass  http://$host:9090$request_uri;
        proxy_set_header tranproxy "true";
    }
}
```

通过 127.0.0.1:9000 端口访问，可以看到返回的请求头中多了 tranproxy=true：

```sh
$ curl -H "Host: echo.example" 127.0.0.1:9000

Hostname: 57e34b409aa1

Pod Information:
    -no pod information available-

Server values:
    server_version=nginx: 1.13.3 - lua: 10008

Request Information:
    client_address=172.17.0.1
    method=GET
    real path=/
    query=
    request_version=1
    request_scheme=http
    request_uri=http://echo.example:8080/

Request Headers:
    accept=*/*
    connection=close
    host=echo.example:9090
    tranproxy=true
    user-agent=curl/7.54.0

Request Body:
    -no body in request-
```

## 实现无感知的本地透明代理

上面的测试因为环境限制，只是验证了透传的功能，这里研究一下怎样实现完全无感知本地 http 透明代理。

目标服务端口为 8080，在请求端设置 nginx，使本地发出的对 8080 端口的访问都经过本地 nginx 代理。

本地 nginx 的配置如下：

```sh
server {
    listen     8080;
    location / {
        resolver 114.114.114.114; 
        proxy_pass  http://$host:8080$request_uri;
        proxy_set_header tranproxy "true";    # 为了验证代理效果，添加了一个请求头
    }
}
```

接下来的关键问题是：怎样将本地发出的到 8080 端口的请求经过 nginx 送出，并且是在客户端无感知的情况下。

借鉴 [istio 的做法][3]，用 iptables 实现，istio 中的 envoy 同时代理流入的请求和流出的请求，我们这里只实现本地的代理，因此只需要设置 outbound 规则。

在 nat 表中添加一条规则链，存放本地代理的规则：

```sh
# 新建一个规则链 
iptables -t nat -N LOCAL_PROXY

# 不代理 nginx 生成的报文，防止出现 nginx 代理 nginx 的死循环
iptables -t nat -A LOCAL_PROXY -m owner --uid-owner nginx -j RETURN

# 将本地生成的目标端口为 8080 的 tcp 报文，重定向到本地的 :8080 监听地址
iptables -t nat -A LOCAL_PROXY -p tcp -m tcp --dport 8080 -j REDIRECT --to-ports 8080
```

启用 LOCAL_PROXY：

```sh
iptables -t nat -A OUTPUT -p tcp -j LOCAL_PROXY
```

然后在本地直接访问目标服务，会发现请求头中多了 tranproxy=true：

```sh
$ curl 172.17.0.21:8080

Hostname: echo-597d89dcd9-m84tq

Pod Information:
    -no pod information available-

Server values:
    server_version=nginx: 1.13.3 - lua: 10008

Request Information:
    client_address=172.17.0.28
    method=GET
    real path=/
    query=
    request_version=1
    request_uri=http://172.17.0.21:8080/
Request Headers:
    accept=*/*
    connection=close
    host=172.17.0.21:8080
    tranproxy=true
    user-agent=curl/7.61.1
```

上面的操作过程已经打包成了镜像，可以直接使用：

```sh
docker run -itd --cap-add NET_ADMIN --name nginx-tranproxy lijiaocn/nginx-tranproxy:0.1 \
    -p 80 \
    -p 8080 \
    -h tranproxy:true \
    -n 114.114.114.114 \
    -n 8.8.8.8
```

在 kubernetes 中使用：

```sh
kubectl -n "Your NameSpace" create -f https://raw.githubusercontent.com/introclass/kubernetes-yamls/master/all-in-one/nginx-tranproxy.yaml
```

## 参考

1. [李佶澳的博客][1]
2. [restart dnsmasq without sudo][2]
3. [服务网格/ServiceMesh 项目 istio 的流量重定向、代理请求过程分析][3]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.stevenrombauts.be/2019/06/restart-dnsmasq-without-sudo/ "Restart dnsmasq without sudo"
[3]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2019/11/01/istio-packet-forward.html "服务网格/ServiceMesh 项目 istio 的流量重定向、代理请求过程分析"

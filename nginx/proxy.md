<!-- toc -->
# 使用 nginx 实现透明代理功能 

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
;echo.example.			IN	A

;; ANSWER SECTION:
echo.example.		0	IN	A	127.0.0.1

;; Query time: 41 msec
;; SERVER: 127.0.0.1#8053(127.0.0.1)
;; WHEN: Wed Oct 30 17:15:03 CST 2019
;; MSG SIZE  rcvd: 57
```

## 配置 nginx

配置nginx

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
        proxy_set_header tranproxy "ture";
    }
}
```

通过 127.0.0.1:9000 端口访问，可以看到返回的请求头中多了 tranproxy=ture：

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
	tranproxy=ture
	user-agent=curl/7.54.0

Request Body:
	-no body in request-
```

## 实现无感知的的透传

上面的测试因为环境限制，只是验证了透传的效果，要实现完全无感知的 http 透传可以这样做：

* 在请求端部署的 nginx 监听 80 端口
* 代理地址为 http://$host:$request_uri
* 域名服务器根据需要设置，这里是 114.114.114.114

```sh
server {
    listen       80;
    listen       [::]:80;
    keepalive_requests  2000;
    keepalive_timeout 60s;

    location / {
        resolver 114.114.114.114; 
        proxy_pass  http://$host:$request_uri;
        proxy_set_header tranproxy "ture";
    }
}


## 参考

1. [李佶澳的博客][1]
2. [Restart dnsmasq without sudo][2]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.stevenrombauts.be/2019/06/restart-dnsmasq-without-sudo/ "Restart dnsmasq without sudo"
z

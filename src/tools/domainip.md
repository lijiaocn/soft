<!-- toc -->
# 域名、IP等相关的工具

## 解析到 127.0.0.1 的公网域名

本站提供一个解析到 127.0.0.1 的公网域名：local.lijiaocn.com。

这个域名的好处是可以通过它访问本地的服务，比方说在本地启动一个echoserver，监听地址为 127.0.0.1:9090，可以用下面的方式访问：

```sh
$ curl local.lijiaocn.com:9090

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
	request_version=1.1
	request_scheme=http
	request_uri=http://local.lijiaocn.com:8080/
...省略...
```

当目标应用不支持填入 IP 地址，或者测试通过域名访问的功能时，这个域名特别有用。

## dnsmasq

试验一些功能时，譬如 [nginx 的透明代理](../nginx/tranproxy.md) 以及 [kubernetes](../k8s/index.md) 的一些功能，需要填写可以通过域名服务器解析的域名。可以在本地用 dnsmasq 搭建一个域名服务解析服务。

### 在 mac 上部署 dnsmasq

* 部署 dnsmasq：

```sh
$ sudo chown -R $(whoami):admin /usr/local
$ brew install dnsmasq
```

* 在 /usr/local/etc/dnsmasq.conf 中添加解析，这里将 echo.example 解析到本地地址 127.0.0.1：

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

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

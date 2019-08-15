# Linux 的 Local 地址的认定

127.0.0.1 默认是 local 地址，其实认定为 local 地址的 IP 是可以设置的，下面的例子将所有 IP 都认定为 local。

在 mangle 表中为要被转发的报文打上标记（1），然后将带有上述标记的报文转发到指定的路由表（100），最后将一批 IP 地址（0.0.0.0/0）设置为路由表（100）的 local：

```sh
iptables -t mangle -I PREROUTING -p udp --dport 5301 -j MARK --set-mark 1
ip rule add fwmark 1 lookup 100
ip route add local 0.0.0.0/0 dev lo table 100
```

设置上述规则后，所有目标端口为 5301 的 UDP 报文，无论目的地址是多少，都被认为是发送到给本机（local）的。这时候如果监听本地地址 0.0.0.0:5301，会收到所有目标端口为 5301 的 udp 报文，无论报文的目的 IP 是不是本地 IP。

## 参考

[1]: https://www.kernel.org/doc/Documentation/networking/tproxy.txt "Transparent proxy support"
[2]: https://powerdns.org/tproxydoc/tproxy.md.html "Linux transparent proxy support"

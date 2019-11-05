<!-- toc -->
# iptables 的报文匹配方法

iptables 规则的意思就是，对满足什么什么条件的报文，做怎样怎样的处理。第一步就是撰写报文的匹配规则，然后设置处理动作。这里收集 iptables 提供的报文匹配方法。

## 基本匹配

`man iptables` 的 PARAMETERS 中列出可以在规则中使用的参数：

```sh
-4, --ipv4
-6, --ipv6
[!] -p, --protocol protocol
[!] -s, --source address[/mask][,...]
[!] -d, --destination address[/mask][,...]
[!] -i, --in-interface name
[!] -o, --out-interface name
[!] -f, --fragment      # 匹配分片报文
                        # This means that the rule only refers to second and 
                        # further IPv4 fragments of fragmented packets.
```

### 支持的协议

-p 后面的 protocol 可以是数字表示的协议号，可以是下面的字符串（0 和 "all" 表示匹配所有协议），或者 /etc/protocols 中列出的协议名：

```sh
tcp, udp, udplite, icmp, icmpv6, esp, ah, sctp, mh, all
```

```sh
$ cat /etc/protocols
# /etc/protocols:
# $Id: protocols,v 1.11 2011/05/03 14:45:40 ovasik Exp $
#
# Internet (IP) protocols
#
#	from: @(#)protocols	5.1 (Berkeley) 4/17/89
#
# Updated for NetBSD based on RFC 1340, Assigned Numbers (July 1992).
# Last IANA update included dated 2011-05-03
#
# See also http://www.iana.org/assignments/protocol-numbers

ip	0	IP		# internet protocol, pseudo protocol number
hopopt	0	HOPOPT		# hop-by-hop options for ipv6
icmp	1	ICMP		# internet control message protocol
igmp	2	IGMP		# internet group management
...省略...
```

## 扩展匹配

基本匹配的规则比较简单，只有协议、源地址、目的地址、接收网卡、发送网卡和分片报文，这几种匹配是远远不够的，iptables 的 -m 参数可以指定扩展模块，实现更复杂、更精细的匹配：

```sh
-m, --match match
```

iptables 提供的扩展模块非常多，种类和用法可以在 `man iptables-extensions` 中看到。

扩展模块分为 MATCH EXTENSIONS 和 TARGET EXTENSIONS 两类，前者用于报文匹配，后者用于报文处理。

下面是 MATCH EXTENSIONS 列表：

```sh
addrtype
ah (IPv6-specific)
ah (IPv4-specific)
bpf
cgroup
cluster
comment
connbytes
connlabel
connlimit
connmark
conntrack
cpu
dccp
devgroup
dscp
dst (IPv6-specific)
ecn
esp
eui64 (IPv6-specific)
frag (IPv6-specific)
hashlimit
hbh (IPv6-specific)
helper
hl (IPv6-specific)
icmp (IPv4-specific)
icmp6 (IPv6-specific)
iprange
ipv6header (IPv6-specific)
ipvs
length
limit
mac
mark
mh (IPv6-specific)
multiport
nfacct
osf
owner
physdev
pkttype
policy
quota
rateest
realm (IPv4-specific)
recent
rpfilter
rt (IPv6-specific)
sctp
set
socket
state
statistic
string
tcp
tcpmss
time
tos
ttl (IPv4-specific)
u32
udp
unclean (IPv4-specific)
```

### 匹配特定用户的报文

istio 中用[下面的方法][2]，对应用户 1337 启动的 envoy 进程创建的报文单独处理：

```sh
-A ISTIO_OUTPUT -m owner --uid-owner 1337 -j RETURN
-A ISTIO_OUTPUT -m owner --gid-owner 1337 -j RETURN
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2019/11/01/istio-packet-forward.html#initcontainers-%E7%94%A8%E9%80%94%E5%88%86%E6%9E%90 "服务网格/ServiceMesh 项目 istio 的流量重定向、代理请求过程分析"

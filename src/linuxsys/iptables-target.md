<!-- toc -->
# iptable 的报文处理动作

设置 [匹配规则](./iptables-match.md) 后，为报文设置处理动作。

报文处理动作用下面是三个参数指定：

```sh
-j, --jump target                  # 使用指定的扩展模块处理报文，处理完成后返回
-g, --goto chain                   # 跳转到另一条规则链处理
-c, --set-counters packets bytes   # 计数器重置
```

报文处理动作主要是用 `-j` 指定，-j 也可以像 -g 一样指向另一条规则链。

另一条规则链我们把它称为子链：

* 如果子链是通过 `-j` 跳转的，子链中的 return 动作会终止子链中处理，返回到父链中继续处理
* 如果子链是通过 `-g` 跳转的，子链中的 return 动作会终止子链中处理，且跳过父链的后续规则

>Unlike the --jump option return will not continue processing in this chain but instead in the chain that called us via --jump.

## 基本动作

```sh
ACCEPT    # 放行报文
DROP      # 丢弃报文
RETURN    # 返回到上一条规则链中继续处理
```

## 扩展动作

大部分处理是用扩展完成的，在 `man iptables-extensions` 中可以看到支持的扩展动作：

```
AUDIT
CHECKSUM
CLASSIFY
CLUSTERIP (IPv4-specific)
CONNMARK
CONNSECMARK
CT
DNAT
DNPT (IPv6-specific)
DSCP
ECN (IPv4-specific)
HL (IPv6-specific)
HMARK
IDLETIMER
LED
LOG
MARK
MASQUERADE
MIRROR (IPv4-specific)
NETMAP
NFLOG
NFQUEUE
NOTRACK
RATEEST
REDIRECT
REJECT (IPv6-specific)
REJECT (IPv4-specific)
SAME (IPv4-specific)
SECMARK
SET
SNAT
SNPT (IPv6-specific)
TCPMSS
TCPOPTSTRIP
TEE
TOS
TPROXY
TRACE
TTL (IPv4-specific)
ULOG (IPv4-specific)
```

### 报文重定向

REDIRECT 能够在 nat 表的 PREROUTING 和 OUTPUT 链中，将报文的目地 IP 修改为接收到这个报文的网卡的 IP，从而将报文重定向到本地的监听地址。

目标端口如果不指定则不修改，复用原始报文的目标端口：

```sh
--to-ports port[-port]   # 将目标端口修改为范围内的数值
--random                 # 为目标端口随机取值
```

示例，[nginx 本地透明代理](../nginx/proxy.md)：

```sh
# 新建一个规则链 
iptables -t nat -N LOCAL_PROXY

# 不代理 nginx 生成的报文，防止出现 nginx 代理 nginx 的死循环
iptables -t nat -A LOCAL_PROXY -m owner --uid-owner nginx -j RETURN

# 将本地生成的目标端口为 8080 的 tcp 报文，重定向到本地的 :8080 监听地址
iptables -t nat -A LOCAL_PROXY -p tcp -m tcp --dport 8080 -j REDIRECT --to-ports 8080
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

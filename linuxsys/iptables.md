<!-- toc -->
# iptables 的使用方法

这里是对 [iptables：Linux的iptables使用][2] 的重新整理、扩充。

## 安装 iptables 工具

在 Debian/Ubuntu 上安装：

```
apt-get update
apt-get install iptables -y
```

## iptables 原理

iptables 根植于 kernel 中的 netfilter 模块，通过 netfilter 在影响正在被内核处理的网络报文，一共有 PREROUTING、FORWARD、POSTROUTING、INPUT 和 OUTPUT 5 个干涉点，每个干涉点可以挂一串的处理规则（Chain）：

```
           INPUT                 OUPUT
             .                     |
            /_\           +--------+
             |           _|_
             +--------+  \ /
                      |   ' 
                      Router --------|> FORWARD
                      .   |                |
                     /_\  +--------+       |
                      |           _|_     _|_
            +---------+           \ /     \ /
            |                      '       ' 
 PKT ---> PREROUTING              POSTROUTING  ---> PKT
```

## iptalbes 的规则管理与作用顺序

iptalbes 的规则通过 5 张表管理，有的表包括所有的干涉点，有的表只包括部分干涉点：

```sh
filter: 
    Chain INPUT
    Chain FORWARD
    Chain OUTPUT

nat:
    Chain PREROUTING
    Chain INPUT
    Chain OUTPUT
    Chain POSTROUTING

mangle:
    Chain PREROUTING
    Chain INPUT
    Chain FORWARD
    Chain OUTPUT
    Chain POSTROUTING

raw:
    Chain PREROUTING
    Chain OUTPUT

security:
    Chain INPUT
    Chain FORWARD
    Chain OUTPUT
```

上面 5 张表中的 Chain 的作用顺序是固定的，贯穿了报文的处理过程，[structure-of-iptables][3] 中有非常详细的说明：

![nf-packet-flow](https://www.lijiaocn.com/img/iptables/nf-packet-flow.png)

* 进入主机的报文:

	raw.PREROUTING -> mangle.PREROUTING -> nat.PREROUTING -> mangle.INPUT -> filter.INPUT 

* 经主机转发的报文:

	raw.PREROUTING -> mangle.PREROUTING -> nat.PREROUTING -> mangle.FORWARD -> filter.FORWARD
	-> mangle.POSTROUTING -> nat.POSTROUTING

* 主机发出的报文:

	raw.OUTPUT -> mangle.OUTPUT -> nat.OUTPUT -> filter.OUTPUT -> mangle.POSTROUTING 
	->nat.POSTROUTING

## 规则格式

规则语法如下：

	rule-specification = [matches...] [target]
	match = -m matchname [per-match-options]
	target = -j targetname [per-target-options]

可以使用的规则参数:

```sh
-4, --ipv4
-6, --ipv6
[!] -p, --protocol protocol
     可以使用:
       1. tcp, udp, udplite, icmp, icmpv6,esp, ah, sctp, mh or the special keyword "all"
       2. 协议号，0等同于"all"
       3. /etc/protocols中列出的协议名
[!] -s, --source address[/mask][,...]
     Address can be either:
         a network name, a hostname, a network IP address (with /mask), or a plain IP address.
         Multiple addresses can be specified, but this will expand to multiple rules (when
         adding with -A), or will cause multiple rules to be deleted (with -D).
[!] -d, --destination address[/mask][,...]
-m, --match match
     不同的模块有不同的参数，在下一节中单独讨论
-j, --jump target
-g, --goto chain
      Unlike the --jump option return will not continue processing in this chain but instead 
      in the chain that called us via --jump
[!] -i, --in-interface name
[!] -o, --out-interface name
[!] -f, --fragment
      This means that the rule only refers to second and further IPv4 fragments of fragmented packets.
      Since there is no way to tell the source or destination ports of  such  a  packet  (or ICMP type), 
      such a packet will not match any rules which specify them.  
      When the "!" argument precedes the "-f" flag, the rule will only match head fragments, or unfragmented packets.
      This option is IPv4 specific, it is not available in ip6tables.
-c, --set-counters packets bytes
      This enables the administrator to initialize the packet and byte counters of a rule 
      (during INSERT, APPEND, REPLACE operations).
```

怎样用 iptables 实现各种效果，见后面章节。

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2014/04/16/linux-net-iptables.html  "iptables：Linux的iptables使用"
[3]: http://www.iptables.info/en/structure-of-iptables.html "structure-of-iptables"

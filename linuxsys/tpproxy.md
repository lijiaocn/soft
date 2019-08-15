# Linux 的透明代理 

TODO: 待试验。

透明代理是 Linux kernel 的功能，用 iptables 或者 nft 命令设置，将发送到地址 A 的报文转发给本地的另一个监听地址，监听改地址的本地进程能够获取到原始的目的地址。

代理报文的通常做法是 REDIRECT，也就是做 NAT，kernel 将报文的目标地址改写为另一个地址，同时将源地址改写为自身的地址，然后维护这两个地址的对应关系。这种方式存在一个明显的弊端，本地接收端不能或者很难获得报文的原始地址。

透明代理是另一种做法，在 kernel 上创建一张路由表，本地接收端（用户态程序）建立监听该路由表中地址的 socket。匹配规则报文被转发给本地接收端后，本地接收端可以用对应的 socket 函数获取原始的目标地址。 

内核文档 [Transparent proxy support][1] 对该功能有简单说明，[Linux transparent proxy support][2] 介绍的更详细。 下面的内容主要来自第二篇文档。

## 准备本地代理进程

本地代理进程是一个用户态进程，创建一个监听 socket 接收新建连接和报文。监听地址可以是任意需要的 IP 地址，但需要注意，如果不是 0.0.0.0，那么需要使用 `IP_TRANSPARENT` 选项。认定为 local 的 IP 地址是可以配置的，见 [Linux 系统知识/local 地址](../linuxsys/localip.md)


本地代理进程从监听 sockert 中读取新建连接，可以自由地将接收的内容转发到另一个地址。

```
Socket s(AF_INET, SOCK_DGRAM, 0);
SSetsockopt(s, IPPROTO_IP, IP_TRANSPARENT, 1);
ComboAddress local("1.2.3.4", 5300);
ComboAddress remote("198.41.0.4", 53);

SBind(s, local);
SSendto(s, "hi!", remote);
```

透明代理能够获得原始地址，指的本地代理程序能够用 socket 函数获取报文的原始地址，TCP 协议用 getsockname() 获取 ：

```
Socket s(AF_INET, SOCK_STREAM, 0);
SSetsockopt(s, IPPROTO_IP, IP_TRANSPARENT, 1);
ComboAddress local("127.0.0.1", 10025);

SBind(s, local);
SListen(s, 128);

ComboAddress remote(local), orig(local);
int client = SAccept(s, remote);
cout<<"Got connection from "<<remote.toStringWithPort()<<endl;

SGetsockname(client, orig);
cout<<"Original destination: "<<orig.toStringWithPort()<<endl;
```

UDP 协议需要用 setsockopt() 设置 IP_RECVORIGDSTADDR，然后从 recvmsg() 收到的 cmsg 中获取，索引为 `IP_ORIGDSTADDR`。


## 设置 iptables 规则

准备好本地代理进程后，还需要设置 iptables 规则，将报文转发给本地代理。

在 iptables 规则中使用名为 TPROXY 的 TARGET，也就是透明代理。下面的规则将目标端口是 25 的 TCP 报文以透明代理的方式发送到了本地地址 127.0.0.1:10025。

```sh
iptables -t mangle -A PREROUTING -p tcp --dport 25 -j TPROXY \
  --tproxy-mark 0x1/0x1 --on-port 10025 --on-ip 127.0.0.1
```

## 参考

[1]: https://www.kernel.org/doc/Documentation/networking/tproxy.txt "Transparent proxy support"
[2]: https://powerdns.org/tproxydoc/tproxy.md.html "Linux transparent proxy support"


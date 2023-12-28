## 说明

[iproute2][2] 是一组在 Linux 中进行网络设备、流量管理的工具。iproute2 的目标是取代历史更悠久的 net-tools 工具集。
iproute2 包含众多命令，在 [iproute2 src: man/man8][3] 中可以看到 iproute2 包含的工具：

```bash
arpd
bridge
lnstat
dcb
devlink
genl
ifstat
ip
rdma
routel
nstat,rtacct
rtmon
ss
tc
tipc
vdpa
```

部分工具包含大量子命令，比如 iproute2 的主要工具 ip 和 tc。以 ip 包含的子命令为例：

```bash
link
address
addrlabel
route
rule
neigh
ntable
tunnel
tuntap
maddress
mroute
mrule
monitor
xfrm
netns
l2tp
tcp_metrics
token
macsec
```

## 参考

1. [李佶澳的博客][1]
2. [iproute2][2]
3. [iproute2 src: man/man8][3]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://wiki.linuxfoundation.org/networking/iproute2 
[3]: https://github.com/iproute2/iproute2/tree/main/man/man8 

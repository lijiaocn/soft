## 说明

`ip link` 用于配置网络设备，包含多个子命令，在 `man ip link` 中可以查看详细说明。

```bash
show
set
add
delete
xstats
afstats
property
```

## ip link show 查看网络设备属性

```bash
$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 02:cd:6a:13:84:49 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:4c:de:3c brd ff:ff:ff:ff:ff:ff
```

ip link show 将网络设备的属性用统一的格式呈现出来。前面三项含义分别是设备编号、设备名称、设备状态标识（<>中的内容）。
之后都是（字段含义 字段值）的格式。以第二个设备为例，ip link 显示的内容含义如下：

```bash
2:                                  在 ip link 输出结果中的编号
enp0s3:                             设备名称
<BROADCAST,MULTICAST,UP,LOWER_UP>   设备状态为 broadcast/multicast/up/lower_up
mtu 65536                           mtu 数值为 65536
qdisc fq_codel                      流量控制策略为 fa_codel
status UP                           设备状态为 UP
mode DEFAULT                        设备模式为 DEFAULT
group default                       设备位于 default 组
qlen 1000                           队列长度 1000
link/ether 02:cd:6a:13:84:49        设备物理地址
brd ff:ff:ff:ff:ff:ff               broadcast 地址
```

### 设备状态标识

(参考链接：[2][2])

Linux 网络设备有很多个状态标识，`man 7 netdeivce` 中列出了已经支持的状态标识。

```bash
            Device flags
IFF_UP            Interface is running.
IFF_BROADCAST     Valid broadcast address set.
IFF_DEBUG         Internal debugging flag.
IFF_LOOPBACK      Interface is a loopback interface.
IFF_POINTOPOINT   Interface is a point-to-point link.
IFF_RUNNING       Resources allocated.
IFF_NOARP         No arp protocol, L2 destination address not set.
IFF_PROMISC       Interface is in promiscuous mode.
IFF_NOTRAILERS    Avoid use of trailers.
IFF_ALLMULTI      Receive all multicast packets.
IFF_MASTER        Master of a load balancing bundle.
IFF_SLAVE         Slave of a load balancing bundle.
IFF_MULTICAST     Supports multicast
IFF_PORTSEL       Is able to select media type via ifmap.
IFF_AUTOMEDIA     Auto media selection active.

IFF_DYNAMIC       The addresses are lost when the interface goes down.
IFF_LOWER_UP      Driver signals L1 up (since Linux 2.6.17)
IFF_DORMANT       Driver signals dormant (since Linux 2.6.17)
IFF_ECHO          Echo sent packets (since Linux 2.6.25)
```

### 流量控制策略 qdisc

(参考链接：[3][3],[4][4])

### 设备分组 group

设备可以加入到指定分组，有一些命令支持按分组进行操作，比如查看指定分组中的设备：

```bash
$ ip link show group default
```

分组信息记录在 `/etc/iproute2/group` 中，可以手动在其中添加：

```bash
$ cat /etc/iproute2/group
# device group names
0	default
```

### 广播地址 brd

(参考链接：[5][5])

## 参考

1. [李佶澳的博客][1]
2. [ip-link-and-ip-addr-output-meaning][2]
3. [linux-traffic-control_configuring-and-managing-networking][3]
4. [tc-fq_codel.8][4]
5. [meaning-of-brd-in-output-of-ip-commands][5]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://unix.stackexchange.com/questions/335077/ip-link-and-ip-addr-output-meaning 
[3]: https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/linux-traffic-control_configuring-and-managing-networking 
[4]: https://man7.org/linux/man-pages/man8/tc-fq_codel.8.html
[5]: https://unix.stackexchange.com/questions/501357/meaning-of-brd-in-output-of-ip-commands

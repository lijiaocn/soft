## 说明

一直想系统学习 linux 网络设备，苦于没有一份能够提供完整知识大纲的资料。既然没有现成的资料，那就自己动手整理吧。

主要参考了一下资料：

* [Guide to IP Layer Network Administration with Linux][2]
* [Linux Advanced Routing & Traffic Control HOWTO][3]
* [iproute2 src: man/man8][4]

另外还参考了大量网上问答的内容，在每个知识点中以参考文献的方式给出了对应的页面链接，这里不一一列出。

## 环境准备

### 用 vagrant 启动三台虚拟机

用 vagrant 启动三台虚拟机，每台虚拟机有两个网卡，一个网卡为 NAT 模式用于联通外网，一个网卡为 Host-only Network 用于网络实验。三台虚拟机规划网址是：

```bash
Gateway：192.168.33.1
Node10：192.168.33.10
Node20：192.168.33.20
Node30：192.368.33.30
```

创建第一台虚拟机 Node10：

```bash
$ mkdir ubuntu10 
$ cd ubuntu10
$ vagrant init ubuntu/focal64
```

vagrant init 会在当前目录中生成文件 Vagrantfile，编辑该文件，将其中的 config.vm.network 设置为如下内容：

```bash
# 配置虚拟机第二块网卡的网络类型以及IP地址
config.vm.network "private_network", ip: "192.168.33.10"
# 配置在 shell 提示符中展示的主机名，只是为了方便通过 shell 提示符确认当前所在的 Node
config.vm.hostname = "Node10"
```

然后启动虚拟机：

```bash
$ vagrant up
```

虚拟机启动结束后，用 vagrant ssh 进入：

```bash
$ vagrant ssh
```

用 ip link 看到有 enp0s3 和 enp0s8 两个网卡：

```bash
$  ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 02:cd:6a:13:84:49 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:4c:de:3c brd ff:ff:ff:ff:ff:ff
```

**用类似的方式创建 Node20 和 Node30，Vagrantfile 中的 IP 地址相应修改成 192.168.33.20 和 192.168.33.30。**

### 实验环境验证

在 Node10 上 Ping Node20 和 Node30，确定网络联通：

```bash
vagrant@Node10$ ping -c 3 192.168.33.20
PING 192.168.33.20 (192.168.33.20) 56(84) bytes of data.
64 bytes from 192.168.33.20: icmp_seq=1 ttl=64 time=0.623 ms
64 bytes from 192.168.33.20: icmp_seq=2 ttl=64 time=1.12 ms
64 bytes from 192.168.33.20: icmp_seq=3 ttl=64 time=1.08 ms

--- 192.168.33.20 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2034ms
rtt min/avg/max/mdev = 0.623/0.940/1.119/0.224 ms
$ ping -c 3 192.168.33.30
PING 192.168.33.30 (192.168.33.30) 56(84) bytes of data.
64 bytes from 192.168.33.30: icmp_seq=1 ttl=64 time=1.43 ms
64 bytes from 192.168.33.30: icmp_seq=2 ttl=64 time=1.02 ms
64 bytes from 192.168.33.30: icmp_seq=3 ttl=64 time=0.879 ms

--- 192.168.33.30 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 0.879/1.109/1.425/0.230 ms
```

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: http://linux-ip.net/html/index.html 
[3]: https://lartc.org/howto/ 
[4]: https://github.com/iproute2/iproute2/tree/main/man/man8 

<!-- toc -->
# 网卡状态

## 多种方式查看网卡信息

`ifconfig`、`netstat`（yum install -y net-tool）或者`ip`（yum install -y iproute2）查看网卡详情。

```sh
$ ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1454
        inet 192.168.7.222  netmask 255.255.0.0  broadcast 10.19.255.255
        ether 52:54:00:33:70:30  txqueuelen 1000  (Ethernet)
        RX packets 2973979082  bytes 1977239275436 (1.7 TiB)
        RX errors 0  dropped 53  overruns 0  frame 0
        TX packets 2157378511  bytes 3340384739019 (3.0 TiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```sh
$ netstat -i
Kernel Interface table
Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
docker0   1426 290827246      0      0 0      309655306      0      0      0 BMRU
eth0      1454 214759910523      0   1296 0      193020803000      0      0      0 BMRU
flannel0  1426 118667062225      0      0 0      110488702382      0   6361      0 MOPRU
lo       65536 184456359      0      0 0      184456359      0      0      0 LRU
lo:1     65536      - no statistics available -                        LRU
vethba02  1426 74175712      0      0 0      80134415      0      0      0 BMRU
```

```sh
$ ip -s -d addr
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1454 qdisc pfifo_fast state UP qlen 1000
    link/ether 52:54:00:33:70:30 brd ff:ff:ff:ff:ff:ff promiscuity 0
    inet 192.168.7.222/16 brd 10.19.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    RX: bytes  packets  errors  dropped overrun mcast
    1977612295437 2974994116 0       53      0       0
    TX: bytes  packets  errors  dropped carrier collsns
    3341796453985 2158430374 0       0       0       0
```

运行状态： RUNNING，LOWER_UP。

MTU：1454。

地址：IP、MAC。

报文收发情况： RX是接收的报文，TX是发送的报文。

```
errors：    错误报文数，校验错误、帧错误等。
dropped：   丢弃的报文数，报文已经进入ring buffer，但是因为内存不足丢弃。
overruns：  超限制报文数，因ring buffer已满而丢弃的报文。
carrier：   双工模式不匹配、电缆故障等导致的错误。
collisions：冲突报文数。
```

## 用sar查看网卡吞吐

用sar查看网卡的吞吐情况，`-n`查看网络状态，`DEV`查看设备状态（除了DEV，还支持：EDEV、 NFS、 NFSD、 SOCK、 IP、 EIP、 ICMP、 EICMP、 TCP、 ETCP、 UDP、 SOCK6、 IP6、 EIP6、 ICMP6、 EICMP6、 UDP6），`1`每秒刷新一次：

```sh
$sar -n DEV 1
Linux 3.10.0-693.11.6.el7.x86_64 (prod-k8s-node-180-92) 	04/22/2019 	_x86_64_	(32 CPU)

02:04:47 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
02:04:48 PM      eth0   1395.00   1186.00    545.40   1902.75      0.00      0.00      0.00
02:04:48 PM  flannel0    381.00    299.00    154.24     83.22      0.00      0.00      0.00
02:04:48 PM   docker0   1177.00   1336.00   1895.27    530.71      0.00      0.00      0.00

```

接收和发送的pps： rxpck/s、txpck/s。

接收和发送的字节数：rxkB/s、txkB/s。

接收和发送的压缩数据包数：rxcmp/s、txcmp/s。

<!-- toc -->
# Linux的连接与协议栈状态的查看方法

## netstat查看连接状态分布

用netstat查看连接状态的分布（用yum安装：yum install -y net-tools）：

```sh
$ netstat -nat|awk '{print awk $NF}'|sort|uniq -c|sort -n
      1 CLOSE_WAIT
      1 FIN_WAIT2
      1 State
      1 established)
      3 LAST_ACK
     30 FIN_WAIT1
    151 LISTEN
    280 TIME_WAIT
   1548 ESTABLISHED
```

## 查看本地临时端口占用情况

```sh
$ netstat -ntp |grep "11.0.110.0"  |awk '{print $4}' |sort |uniq |wc
  18171   18171  308907
```

查看本地源端口范围：

```sh
$ sysctl  net.ipv4.ip_local_port_range  
$ cat /proc/sys/net/ipv4/ip_local_port_range
```

设置本地源端口范围：

```sh
sysctl -w net.ipv4.ip_local_port_range="1024 64000"
```

临时端口不足时，Nginx会报下面的错误：

```
 (99: Cannot assign requested address) while connecting to upstream, client: 192.168.0.2, server: localhost, request: "GET / HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "192.168.0.30"
```


## netstat/ss查看单个连接状态

用netstat查看详细连接，-t查看tcp连接，-u查看udp连接，-p显示对应进程：

```sh
$ netstat -ntp
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 11.0.157.1:32888        11.0.157.6:8080         TIME_WAIT   -
tcp        0      0 11.0.157.1:45894        11.0.157.4:8080         TIME_WAIT   -
tcp        0      0 11.0.157.1:35826        11.0.157.9:8080         TIME_WAIT   -
tcp        0      0 11.0.157.1:57348        11.0.157.3:8080         TIME_WAIT   -
tcp        0      0 127.0.0.1:54728         127.0.0.1:10248         TIME_WAIT   -
```

ss命令是netstat的替代命令：

```sh
$ ss -ntp
State      Recv-Q Send-Q     Local Address:Port         Peer Address:Port
ESTAB      0      0           10.19.180.92:10250        10.19.14.106:38292     users:(("kubelet",pid=15242,fd=41))
ESTAB      0      0           10.19.180.92:35154        10.19.61.146:2379      users:(("flanneld",pid=1042,fd=10))
ESTAB      0      0           10.19.180.92:45380        10.19.162.39:6443      users:(("kube-proxy",pid=14013,fd=8))
```

当套接字是连接状态时，Established：

**Recv-Q**：套接字接收队列中还没有被应用程序读取的字节数。

**Send-Q**：套接字发送队列中还没有被远端主机确认的字节数。

当套接字是监听状态时，Listening，显示的半连接状态，即服务端收到客户端SYN，还没有完成三次握手的TCP连接：

**Recv-Q**：TCP协议栈中的半连接队列长度的当前值，syn backlog当前值。

**Send-Q**：TCP协议栈中的半连接队列长度的最大值，syn backlog最大值。

套接字是Established状态时，如果Recv-Q和Send-Q不为零，说明接收的数据无法被应用程序及时读取，以及应用的数据无法及时发送，是异常情况。

>问题：如果是UDP协议呢？Rece-Q和Send-Q的含义是什么？同Established。

## netstat查看协议栈状态

`netstat -s`打印出整个协议栈的状态，按照协议类型分开展示：

```sh
$ netstat -s
Ip:
    3635473208 total packets received
    2855004908 forwarded
    0 incoming packets discarded
    769088666 incoming packets delivered
    3701098707 requests sent out
    44 dropped because of missing route
Icmp:
    100236 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 995
        redirects: 4
        echo requests: 99229
        echo replies: 8
    114887 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 15658
        echo replies: 99229
IcmpMsg:
        InType0: 8
        InType3: 995
        InType5: 4
        InType8: 99229
        OutType0: 99229
        OutType3: 15658
Tcp:
    3716853 active connections openings
    1228040 passive connection openings
    570 failed connection attempts
    610 connection resets received
    17 connections established
    33850465 segments received
    43122661 segments send out
    15575 segments retransmited
    4 bad segments received.
    1018 resets sent
Udp:
    735086502 packets received
    6 packets to unknown port received.
    33475 packet receive errors
    797838180 packets sent
    33475 receive buffer errors
    0 send buffer errors
UdpLite:
TcpExt:
    28250 invalid SYN cookies received
    3961 resets received for embryonic SYN_RECV sockets
    21371 packets pruned from receive queue because of socket buffer overrun
    72874448 TCP sockets finished time wait in fast timer
    1396 packets rejects in established connections because of timestamp
    6946924 delayed acks sent
    128318 delayed acks further delayed because of locked socket
    Quick ack mode was activated 49028 times
    1292037 packets directly queued to recvmsg prequeue.
    33405 bytes directly in process context from backlog
    566548477 bytes directly received in process context from prequeue
    420652665 packet headers predicted
    1255272 packets header predicted and directly queued to user
    285092354 acknowledgments not containing data payload received
...
```

##  限制半开连接

半开连接最大数量：

```sh
sysctl net.ipv4.tcp_max_syn_backlog
net.ipv4.tcp_max_syn_backlog = 256
```

限制半开连接建立速度：

```sh
# 限制 syn 并发数为每秒 1 次
$ iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT

# 限制单个 IP 在 60 秒新建立的连接数为 10
$ iptables -I INPUT -p tcp --dport 80 --syn -m recent --name SYN_FLOOD --update --seconds 60 --hitcount 10 -j REJECT
```

启动synccookie：

```sh
sysctl -w net.ipv4.tcp_syncookies=1
net.ipv4.tcp_syncookies = 1
```


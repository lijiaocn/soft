<!-- toc -->
# 网络性能指标

**带宽**: 最大传输速率，单位b/s（比特/秒）。

**吞吐量**: 单位时间内传输的数据量，b/s或者B/s。

**延时**：请求发出时与对方收到时的间隔时间，不同场景下含义不同，例如TCP握手延迟，数据包往返延迟（RTT）。

**PPS**: 每秒传输的报文数（不论报文大小），硬件交换机通常可以达到线速，Linux服务器一般达不到。

**并发连接数**: tcp连接数量。

**丢包率**: 丢失报文占比。

**重传率**: 重传报文占比。 

## hping3检测延迟

hping3的用法：-c 表示发送3次请求，-S 表示设置TCP SYN，-p 表示端口号为80（yum安装：yum install -y hping3）。

```sh
$ hping3 -c 3 -S -p 80 baidu.com
HPING baidu.com (eth0 123.125.115.110): S set, 40 headers + 0 data bytes
len=46 ip=123.125.115.110 ttl=51 id=47908 sport=80 flags=SA seq=0 win=8192 rtt=20.9 ms
len=46 ip=123.125.115.110 ttl=51 id=6788  sport=80 flags=SA seq=1 win=8192 rtt=20.9 ms
len=46 ip=123.125.115.110 ttl=51 id=37699 sport=80 flags=SA seq=2 win=8192 rtt=20.9 ms

--- baidu.com hping statistic ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 20.9/20.9/20.9 ms
```

## traceroute检测延迟 

traceroute的用法：--tcp 表示使用 TCP 协议，-p 表示端口号，-n 表示不对结果中的 IP 地址执行反向域名解析（yum安装：yum install -y traceroute）。

```sh
$ traceroute --tcp -p 80 -n baidu.com
traceroute to baidu.com (123.125.115.110), 30 hops max, 60 byte packets
 1  * * *
 2  * * *
 3  * * *
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  * * *
 9  * * *
10  * * *
11  * * *
12  * * *
13  * * *
14  123.125.115.110  20.684 ms *  20.798 ms
```

## pktgen发包

pktgen是内核自带的发包工具，[networking/pktgen.txt](https://www.kernel.org/doc/Documentation/networking/pktgen.txt)，Intel提供了一个基于DPDK的同名工具[The Pktgen Application](https://pktgen-dpdk.readthedocs.io/en/latest/)。






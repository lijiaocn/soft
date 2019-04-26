<!-- toc -->
# Linux的netfilter

![net filter](/img/linux/net-filter.png)

## 连接跟踪表参数

连接跟踪表tcp超时时间，默认120秒：

```sh
sysctl net.netfilter.nf_conntrack_tcp_timeout_time_wait
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
```

连接跟踪表大小设置：

```sh
$ sysctl net.netfilter.nf_conntrack_max=131072       //跟踪表的总容量
$ sysctl net.netfilter.nf_conntrack_buckets=65536    //跟踪表的桶数量
```

连接跟踪表内存开销计算，连接跟踪对象大小为376，链表项大小为16：

```sh
nf_conntrack_max* 连接跟踪对象大小 +nf_conntrack_buckets* 链表项大小
= 1000*376+65536*16 B
= 1.4 MB
```

## conntrack查看连接跟踪表的内容

conntrack查看连接跟踪表的内容， -L 表示列表，-o 表示以扩展格式显示：

```sh
$ conntrack -L -o extended
ipv4     2 tcp      6 7 TIME_WAIT src=192.168.0.2 dst=192.168.0.96 sport=51744 dport=8080 src=172.17.0.2 dst=192.168.0.2 sport=8080 dport=51744 [ASSURED] mark=0 use=1
ipv4     2 tcp      6 6 TIME_WAIT src=192.168.0.2 dst=192.168.0.96 sport=51524 dport=8080 src=172.17.0.2 dst=192.168.0.2 sport=8080 dport=51524 [ASSURED] mark=0 use=1
```

## 统计总的连接跟踪数

```sh
$ conntrack -L -o extended | wc -l
14289
```

## 统计TCP协议各个状态的连接跟踪数

```sh
$ conntrack -L -o extended | awk '/^.*tcp.*$/ {sum[$6]++} END {for(i in sum) print i, sum[i]}'
SYN_RECV 4
CLOSE_WAIT 9
ESTABLISHED 2877
FIN_WAIT 3
SYN_SENT 2113
TIME_WAIT 9283
```

## 统计各个源IP的连接跟踪数

```sh
$ conntrack -L -o extended | awk '{print $7}' | cut -d "=" -f 2 | sort | uniq -c | sort -nr | head -n 10
  14116 192.168.0.2
    172 192.168.0.96
```

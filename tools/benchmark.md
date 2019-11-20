<!-- toc -->
# 性能测试工具

## iperf 测试网络传输性能

[iperf][2] 是一个简单常用的网络传输性能测试工具，分为服务端和客户端，[iperf-doc][3]。

安装：

```sh
# for CentOS
yum install -y epel-release
yum install -y iperf
```

启动 Server 端：

```sh
$ iperf -p 5001 -s

# -s: server模式
# -p: 监听端口，默认5001
```

服务端可以用容器启动：

```sh
$ docker run -p 5001:5001 lijiaocn/iperf-server:1.0
```

启动 Client 端：

```sh
$ iperf -p 5001 -c 192.168.10.2  -l 1M -t 120

#-p: server 端口，默认 5001
#-c: server 地址
#-l: 每次发送的数据的长度，默认 tcp 是 128K，UDP 是 8K
#-t: 持续的时间
```

## netperf 更精细的网络传输测试

[NetPerf][5] 能够测试更多场景，一个很强大的网络性能测试工具，[netperf doc][7] 中有介绍。

下载源代码，编译安装：

```sh
$ yum install -y gcc make git texinfo
$ git clone https://github.com/HewlettPackard/netperf.git
$ cd netperf
$ ./autogen.sh
$ ./configure --prefix=/usr/local/
$ sudo make install
```

启动服务端：

```sh
$ netserver -4 -p 7777
```

启动客户端，客户端指定测试类型：

```sh
$ netperf -4 -H 127.0.0.1 -p 7777 -t TCP_RR 
```

netperf 的 -t 参数支持很多场景，[netperf/cases][8] 中整理了一部分。

## wrk 测试 http 服务性能

[wrk] 是一个特别高效的 http 测试工具，推荐使用。[怎样压测 Web 应用的性能？压测工具与测量、分析方法][10] 中有更多工具 。

```sh
$ git clone https://github.com/wg/wrk.git
$ cd wrk 
$ make
```

使用方法：

```sh
$ ./wrk
Usage: wrk <options> <url>
  Options:
    -c, --connections <N>  Connections to keep open
    -d, --duration    <T>  Duration of test
    -t, --threads     <N>  Number of threads to use

    -s, --script      <S>  Load Lua script file
    -H, --header      <H>  Add header to request
        --latency          Print latency statistics
        --timeout     <T>  Socket/request timeout
    -v, --version          Print version details

  Numeric arguments may include a SI unit (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h)
```

测试用例：

```sh
$ ./wrk -t 32 -c 64 -d 60s -H "Host: webshell.com" http://172.16.129.4/ping
```

## ghz 测试 grpc 服务性能

[Grpc性能压测方法：用ghz进行压测](https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2019/02/22/grpc-benchmark-method.html)

## 参考

1. [李佶澳的博客][1]
2. [iperf、netperf][2]
3. [siege][3]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2016/04/08/network-benchmark.html "iperf、netperf等网络性能测试工具的使用"
[3]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/11/08/kong-features-06-production-and-benchmark.html#%E7%94%A8siege%E8%BF%9B%E8%A1%8C%E6%B5%8B%E8%AF%95 "siege"
[4]: https://iperf.fr/iperf-doc.php "iperf-doc.php"
[5]: https://hewlettpackard.github.io/netperf/ "Netperf Homepage"
[6]: https://github.com/HewlettPackard/netperf "Github NetPerf"
[7]: https://hewlettpackard.github.io/netperf/doc/netperf.html "Care and Feeding of Netperf 2.7.X"
[8]: https://github.com/lijiaocn/workspace/tree/master/net-benckmark/netperf/cases "netperf/cases"
[9]: https://www.lijiaocn.com/%E6%96%B9%E6%B3%95/2018/11/02/webserver-benchmark-method.html#wrk "wrk"
[10]: https://www.lijiaocn.com/%E6%96%B9%E6%B3%95/2018/11/02/webserver-benchmark-method.html "怎样压测Web应用的性能？压测工具与测量、分析方法"

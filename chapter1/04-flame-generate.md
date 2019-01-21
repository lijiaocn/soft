# 火焰图生成命令

火焰图的生成分为采集数据和生成图片两个过程，这两个过程中有多种工具可选。

采集数据的工具可以根据自己实际情况、喜好以及熟悉程度选取。

生成图片时，一般使用[FlameGraph](https://github.com/brendangregg/FlameGraph)就足够了。

## 用perf采集调用栈（On CPU）

CPU Flame Graphs的[Instructions](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html#Instructions)章节中给出了用perf采集的方法。

```bash
pid=目标进程
perf record -p $pid -F 99 -a -g 
```

采集的数据位于当前目录中的perf.data文件中，用`perf script`转换成可读格式：

```bash
perf script > perf.bt
```

用[FlameGraph](https://github.com/brendangregg/FlameGraph)中的`stackcollapse-perf.pl`和`flamegraph.pl`生成火焰图：

```bash
./FlameGraph/stackcollapse-perf.pl  perf.bt  >perf.cbt
./FlameGraph/flamegraph.pl perf.cbt > perf.svg
```

## 用stapxx采集调用栈（On CPU)

>注意：在采集envoy的调用栈（On CPU）时，发现perf效果更好，采集得更精细和准确。

[stapxx](https://github.com/openresty/stapxx)是OpenResty作者章宜春开发的一套脚本，简化了用SystemTAP采集数据的操作。

需要事先安装SystemTAP以及kernel的debuginfo包，具体安装安装过程参考[Web开发平台OpenResty（三）：火焰图性能分析](https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/11/02/openresty-study-03-frame-md.html)，这里不赘述。

安装了SystemTAP等依赖包之后，下载stapxx脚本：

	git clone https://github.com/openresty/stapxx
	export PATH=$PATH:`pwd`/stapxx

`stapxx/samples`目录中提供了很多脚本，大部分是用于openresty的，但也有很多通用的脚本，譬如`sample-bt.sxx`、`cpu-hogs.sxx`等。

```
$ ls stapxx/samples/
cpu-hogs.sxx                   ngx-lj-gc-speed.sxx              ngx-rps.sxx
cpu-robbers.sxx                ngx-lj-trace-exits.sxx           ngx-single-req-latency.sxx
ctx-switches.sxx               ngx-lua-count-timers.sxx         ngx-slow-purge-reqs.sxx
epoll-et-lt.sxx                ngx-lua-exec-time.sxx            ngx-sr-trunc.sxx
epoll-loop-blocking-distr.sxx  ngx-lua-shdict-info.sxx          ngx-timeout-settings.sxx
epoll-loop-blocking.sxx        ngx-lua-shdict-writes.sxx        ngx-upstream-err-log.sxx
epoll-loop-blocking-vfs.sxx    ngx-lua-slow-udp-query.sxx       ngx-upstream-latency.sxx
func-latency-distr.sxx         ngx-lua-tcp-recv-time.sxx        ngx-upstream-next.sxx
lj-bucket-depth-accessed.sxx   ngx-lua-tcp-total-recv-time.sxx  ngx-upstream-post-conn.sxx
lj-find-str.sxx                ngx-lua-udp-recv-time.sxx        ngx-zlib-deflate-time.sxx
lj-gc-objs.sxx                 ngx-lua-udp-total-recv-time.sxx  ngx-zlib-total-deflate-time.sxx
lj-gc.sxx                      ngx-lua-useless-cosockets.sxx    openssl-handshake-diagnosis.sxx
lj-lua-bt.sxx                  ngx-open-file-cache-misses.sxx   probe-rate.sxx
lj-lua-stacks.sxx              ngx-orig-resp-body-len.sxx       sample-bt-leaks.sxx
lj-str-tab.sxx                 ngx-pcre-dist.sxx                sample-bt-leaks-wrapalloc.sxx
lj-trace-exit-rate.sxx         ngx-pcre-top.sxx                 sample-bt.sxx
lj-vm-states.sxx               ngx-req-latency-distr.sxx        slow-vfs-reads.sxx
luajit21-gc64                  ngx-req-pool-allocs.sxx          unix-stream-socks.sxx
ngx-conn-timers.sxx            ngx-req-pool-size.sxx            vfs-page-cache-misses.sxx
ngx-count-conns.sxx            ngx-rewrite-latency-distr.sxx    zlib-deflate-chunk-size.sxx
```

这些脚本的用法在[README](https://github.com/openresty/stapxx#table-of-contents)中有说明，这里只演示部分脚本的用法。

采集指定进程的调用栈：

```bash
pid=目标进程
./stapxx/samples/sample-bt.sxx --arg time=20 --skip-badvars -D  MAXSKIPPED=100000 -D MAXMAPENTRIES=100000 -x $pid >resty.bt
```

用[FlameGraph](https://github.com/brendangregg/FlameGraph)中的`stackcollapse-stap.pl`和`flamegraph.pl`生成火焰图：

```bash
git clone https://github.com/brendangregg/FlameGraph.git
./FlameGraph/stackcollapse-stap.pl resty.bt  >resty.cbt
./FlameGraph/flamegraph.pl resty.cbt > resty.svg
```


<!-- toc -->
# Linux系统的IO状态

## dstat查看系统整体IO状态 

dstat同时展示CPU、磁盘IO和网络IO的状态，以及系统终端和上下文切换数量，用yum直接安装即可：

```sh
yum install -y dstat
```

dstat的运行效果如下，1表示每1秒输出一次，10表示一共输出10组数据：

![dstat输出](/img/linux/dstat.png)

## iostat查看磁盘设备IO状态

iostat呈现的数据来自`/proc/diskstats`，`-d -x` 显示所有磁盘的I/O情况。

```sh
$ iostat -d -x 1
Device:    rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda          0.00     0.01    0.43    0.19    74.76     1.59   243.28     0.06    4.63    1.67   11.27  97.45   6.12
vdb          0.00     0.00    0.72    0.01   146.07     0.19   402.12     0.00    1.43    1.26   11.25   0.15   0.01

Device:    rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda          0.00     0.00    1.00    0.00     8.00     0.00    16.00     0.00    4.00    4.00    0.00   4.00   0.40
vdb          0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

Device:    rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda          0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
vdb          0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
```

rrqm/s： 每秒合并读请求数

r/s：每秒发送给磁盘的读请求数（合并后） 

rkB/s：每秒从磁盘读取的数据量，单位KB

r_await： 读响应时间

wrqm/s： 合并写请求速率

w/s：每秒发送给磁盘的写请求数（合并后）

wkB/s：每秒向磁盘写入的数据量，单位KB

w_await：写响应时间

avgqu-sz/aqu-sz：平均请求队列长度

svctm： 推断的处理I/O请求需要的平均时间，单位是毫秒

%util：磁盘处理I/O的时间占比，即使用率，使用率100%，说明I/O操作多（不等于磁盘饱和，饱和是不能再接收新的读写)

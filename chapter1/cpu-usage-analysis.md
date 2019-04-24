<!-- toc -->
# CPU使用分析

CPU使用分析主要就是分析CPU的使用率，看看那些进程占用的CPU资源比较多。

## 用pidstat查看进程的CPU使用率

`pidstat`命令可以显示每个进程的在不同CPU状态中耗费的时间的百分比(1，每秒显示一次；-p，指定进程，如果不指定，显示所有进程)：

```bash
$ pidstat  1 -p  27936
Linux 3.10.0-693.11.6.el7.x86_64 (10.10.64.58) 	12/04/2018 	_x86_64_	(4 CPU)

05:00:59 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
05:01:00 PM    99     27936    0.00    0.00    0.00    0.00     0  openresty
05:01:01 PM    99     27936    0.00    0.00    0.00    0.00     0  openresty
05:01:02 PM    99     27936    0.00    0.00    0.00    0.00     0  openresty
```

## 用perf分析CPU使用率高

`perf top`显示占用CPU时间最多的函数或者指令：

```bash
$ perf top
Samples: 3K of event 'cpu-clock', Event count (approx.): 903937500
Overhead  Shared Object          Symbol
   8.69%  perf                   [.] symbols__insert
   5.33%  perf                   [.] rb_next
   3.41%  [kernel]               [k] _raw_spin_unlock_irqrestore
   3.12%  libc-2.17.so           [.] __memcpy_ssse3_back
   2.40%  [kernel]               [k] finish_task_switch
   2.40%  libc-2.17.so           [.] __strchr_sse42
   2.08%  libelf-0.168.so        [.] gelf_getsym
...省略后续内容...
```

## 用perf report查看cpu事件占比

用`perf record`将采样数据保存，然后用`perf record`查看，或者直接用下面的命令一次完成，`-a`查看所有cpu： 

```sh
perf record -ag  -- sleep 15;perf report
```

perf report中显示，stress进程的cpu事件占比是77%，它大量调用了随机数生成函数random()：

![perf-report](/img/perf-report-1.png)

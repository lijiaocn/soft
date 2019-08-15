<!-- toc -->
# CPU的运行状态

CPU一共有多种工作状态，其中只有`idle`状态是空闲状态。

`top`命令显示过去2秒（2s是默认值，可以用-d参数修改）中，每个CPU在每个状态中停留的时间比例：

	%Cpu(s): 14.5 us,  0.3 sy,  0.0 ni, 85.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

`us`是CPU在用户态的时间比例，`sy`是CPU在内核态的时间比例，其它指标含义如下：

	us, user    : time running un-niced user processes
	sy, system  : time running kernel processes
	ni, nice    : time running niced user processes
	id, idle    : time spent in the kernel idle handler
	wa, IO-wait : time waiting for I/O completion
	hi : time spent servicing hardware interrupts
	si : time spent servicing software interrupts
	st : time stolen from this vm by the hypervisor

在`man top`和中可以找到这些说明。另外`man proc`给出了更多状态说明：

```
user   (1) Time spent in user mode.

nice   (2) Time spent in user mode with low priority (nice).

system (3) Time spent in system mode.

idle   (4) Time spent in the idle task. 
           This value should be USER_HZ times the second entry in the /proc/uptime pseudo-file.

iowait (since Linux 2.5.41)
       (5) Time waiting for I/O to complete.

irq (since Linux 2.6.0-test4)
       (6) Time servicing interrupts.

softirq (since Linux 2.6.0-test4)
       (7) Time servicing softirqs.

steal (since Linux 2.6.11)
       (8) Stolen time, which is the time spent in other operating systems when running in a virtualized environment

guest (since Linux 2.6.24)
       (9) Time spent running a virtual CPU for guest operating systems under the control of the Linux kernel.

guest_nice (since Linux 2.6.33)
       (10) Time spent running a niced guest (virtual CPU for guest operating systems under the control of the Linux kernel).
```

[Understanding CPU Steal Time - when should you be worried?](http://blog.scoutapp.com/articles/2013/07/25/understanding-cpu-steal-time-when-should-you-be-worried)中详细介绍了`steal time`，如果steal time占比持续20分钟超过10%，vm性能可能受到了显著影响。

## CPU使用率的计算

CPU使用率计算方法是：1-空闲时间/CPU总时间。

## CPU处于每种状态的时间

top中会显示CPU在每个状态的时间占比，这些数据来自于`/proc/stat`，这个文件中记录每个CPU在每种模式下耗费的时间分片数量，第一行是所有CPU的数值的累加：

```bash
$ cat /proc/stat
cpu  2295737 1270 903726 238996130 61210 0 90996 27778 0 0
cpu0 435456 334 213700 59922519 15305 0 18523 6715 0 0
cpu1 626136 317 229817 59638019 15702 0 35754 7526 0 0
cpu2 635052 320 237094 59685588 15354 0 18630 6930 0 0
cpu3 599092 297 223113 59750002 14847 0 18087 6605 0 0
...
```

注意：`/proc/stat`中数值的单位不是秒/毫秒，而是时间分片的个数：

	The amount of time, measured in units of USER_HZ (1/100ths of a second on most architectures,
	use sysconf(_SC_CLK_TCK) to obtain the right value), that the system spent in vari‐

每个时间分片时长是1/100秒。

## 用户态与内核态的详细说明

用户态和内核态其实是CPU的工作模式，一些特殊的指令可以让CPU在用户态和内核态的切换。
CPU在内核态时，操作的是内核中的数据，用户态时操作的是用户态数据。

用户态进程是通过发起系统调用，进入到内核态的。

系统调用可以理解为内核提供的功能接口，用户态程序发起系统调用的时候，CPU转为内核态，完成了相关操作后，再切换回用户态。

## 参考

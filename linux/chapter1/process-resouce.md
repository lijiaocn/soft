<!-- toc -->
# Linux查看指定进程占用的资源

查看指定进程占用的资源。

## pidstat查看进程的CPU使用率

`pidstat`命令可以显示每个进程的在不同CPU状态中耗费的时间的百分比(1，每秒显示一次；-p，指定进程，如果不指定，显示所有进程)：

```bash
$ pidstat  1 -p  27936
Linux 3.10.0-693.11.6.el7.x86_64 (10.10.64.58) 	12/04/2018 	_x86_64_	(4 CPU)

05:00:59 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
05:01:00 PM    99     27936    0.00    0.00    0.00    0.00     0  openresty
05:01:01 PM    99     27936    0.00    0.00    0.00    0.00     0  openresty
05:01:02 PM    99     27936    0.00    0.00    0.00    0.00     0  openresty
```

## pidstat查看进程的IO情况

pidstat，`-d`表示展示IO信息，1表示每秒输出一次，使用-p指定进程号，下面间隔 1 秒输出 3 组数据：

```sh
$ pidstat -d -p 4344 1 3
06:38:50      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
06:38:51        0      4344      0.00      0.00      0.00       0  app
06:38:52        0      4344      0.00      0.00      0.00       0  app
06:38:53        0      4344      0.00      0.00      0.00       0  app
```

## pidstat查看进程的切换情况

`pidstat -w -p 进程号`可以显示指定进程的上下文切换情况：

```bash
$ pidstat -w 2 -p 426
Linux 4.20.12-1.el7.elrepo.x86_64 (10.10.64.58) 	04/29/2019 	_x86_64_	(4 CPU)

03:01:12 PM   UID       PID   cswch/s nvcswch/s  Command
03:01:14 PM    99       426      2.99      0.00  openresty
03:01:16 PM    99       426      2.50      0.00  openresty
03:01:18 PM    99       426      3.00      0.00  openresty
03:01:20 PM    99       426      2.00      0.00  openresty
```

`-w`参数的作用是显示进程切换状态，每一列的含义如下（可以在`man pidstat`中找到）：

	 -w     Report task switching activity (kernels 2.6.23 and later only).  
	        The following values may be displayed:
	  UID
	         The real user identification number of the task being monitored.
	
	  USER
	         The name of the real user owning the task being monitored.
	
	  PID
	         The identification number of the task being monitored.
	
	  cswch/s
	         Total number of voluntary context switches the task made per second.  
	         A voluntary context switch occurs when a task blocks because it requires 
	         a resource that is unavailable.
	
	  nvcswch/s
	         Total number of non voluntary context switches the task made per second.  
	         A involuntary context switch takes place when a task executes for the duration 
	         of its  time  slice  and then is forced to relinquish the processor.
	
	  Command
	         The command name of the task.

需要注意自愿切换（cswch/s，voluntary context switches）和非自愿切换（nvcswch/s，non voluntary context switches）的区别。前者是因为需要的资源没有准备好，主动让出CPU发生的切换，后者是进程分配的时间片已经用完，被调度器强制切换。

pidstats -w显示的是进程的状态，如果要将线程一并显示出来，需要再加一个-t参数：

```bash
$ pidstat -wt
Average:      UID      TGID       TID   cswch/s nvcswch/s  Command
Average:        0         3         -      3.92      0.00  ksoftirqd/0
Average:        0         -         3      3.92      0.00  |__ksoftirqd/0
Average:        0         9         -     45.59      0.00  rcu_sched
Average:        0         -         9     45.59      0.00  |__rcu_sched
Average:        0        13         -      1.96      0.00  ksoftirqd/1
Average:        0         -        13      1.96      0.00  |__ksoftirqd/1
Average:        0        17         -      0.49      0.00  migration/2
```

另外pidstat还有一个`-u`参数，可以一并输出进程和线程（加-t）的CPU使用情况：

```bash
[root@10.10.64.58 ~]#  pidstat -wt -u
Linux 3.10.0-693.11.6.el7.x86_64 (10.10.64.58) 	12/04/2018 	_x86_64_	(4 CPU)

04:21:56 PM   UID      TGID       TID    %usr %system  %guest    %CPU   CPU  Command
04:21:56 PM     0         1         -    0.02    0.01    0.00    0.03     0  systemd
04:21:56 PM     0         -         1    0.02    0.01    0.00    0.03     0  |__systemd
04:21:56 PM     0         2         -    0.00    0.00    0.00    0.00     0  kthreadd
04:21:56 PM     0         -         2    0.00    0.00    0.00    0.00     0  |__kthreadd

04:21:56 PM   UID      TGID       TID   cswch/s nvcswch/s  Command
04:21:56 PM     0         1         -      1.75      0.00  systemd
04:21:56 PM     0         -         1      1.75      0.00  |__systemd
04:21:56 PM     0         2         -      0.01      0.00  kthreadd
04:21:56 PM     0         -         2      0.01      0.00  |__kthreadd
04:21:56 PM     0         3         -      2.53      0.00  ksoftirqd/0
```

## lsof查看进程打开的文件

lsof查看进程打开的文件，-p指定进程号，注意必须是进程号，不能是线程号：

```sh
$ lsof -p 18940 
COMMAND   PID USER   FD   TYPE DEVICE  SIZE/OFF    NODE NAME 
python  18940 root  cwd    DIR   0,50      4096 1549389 / 
python  18940 root  rtd    DIR   0,50      4096 1549389 / 
… 
python  18940 root    2u   CHR  136,0       0t0       3 /dev/pts/0 
python  18940 root    3w   REG    8,1 117944320     303 /tmp/logtest.txt 
```

## strace查看进程发起的系统调用

strace跟踪进程的系统调用，-f 表示跟踪子进程和子线程，-T 表示显示系统调用的时长，-tt 表示显示跟踪时间：

```sh
$ strace -f -T -tt -p 9085
[pid  9085] 14:20:16.826131 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 65, NULL, 8) = 1 <0.000055>
[pid  9085] 14:20:16.826301 read(8, "*2\r\n$3\r\nGET\r\n$41\r\nuuid:5b2e76cc-"..., 16384) = 61 <0.000071>
[pid  9085] 14:20:16.826477 read(3, 0x7fff366a5747, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000063>
[pid  9085] 14:20:16.826645 write(8, "$3\r\nbad\r\n", 9) = 9 <0.000173>
[pid  9085] 14:20:16.826907 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 65, NULL, 8) = 1 <0.000032>
[pid  9085] 14:20:16.827030 read(8, "*2\r\n$3\r\nGET\r\n$41\r\nuuid:55862ada-"..., 16384) = 61 <0.000044>
[pid  9085] 14:20:16.827149 read(3, 0x7fff366a5747, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000043>
[pid  9085] 14:20:16.827285 write(8, "$3\r\nbad\r\n", 9) = 9 <0.000141>
[pid  9085] 14:20:16.827514 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 64, NULL, 8) = 1 <0.000049>
[pid  9085] 14:20:16.827641 read(8, "*2\r\n$3\r\nGET\r\n$41\r\nuuid:53522908-"..., 16384) = 61 <0.000043>
[pid  9085] 14:20:16.827784 read(3, 0x7fff366a5747, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000034>
[pid  9085] 14:20:16.827945 write(8, "$4\r\ngood\r\n", 10) = 10 <0.000288>
[pid  9085] 14:20:16.828339 epoll_pwait(5, [{EPOLLIN, {u32=8, u64=8}}], 10128, 63, NULL, 8) = 1 <0.000057>
[pid  9085] 14:20:16.828486 read(8, "*3\r\n$4\r\nSADD\r\n$4\r\ngood\r\n$36\r\n535"..., 16384) = 67 <0.000040>
[pid  9085] 14:20:16.828623 read(3, 0x7fff366a5747, 1) = -1 EAGAIN (Resource temporarily unavailable) <0.000052>
[pid  9085] 14:20:16.828760 write(7, "*3\r\n$4\r\nSADD\r\n$4\r\ngood\r\n$36\r\n535"..., 67) = 67 <0.000060>
[pid  9085] 14:20:16.828970 fdatasync(7) = 0 <0.005415>
[pid  9085] 14:20:16.834493 write(8, ":1\r\n", 4) = 4 <0.000250>
```

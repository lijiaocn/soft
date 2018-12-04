<!-- toc -->
# 理解进程 

要理解CPU负载，需要先知道进程，知道进程的几个状态。

## 进程的分类

[Understanding Linux Process States](https://access.redhat.com/sites/default/files/attachments/processstates_20120831.pdf)对Linux的进程（Process）做了很详细的介绍。

在Linux中，进程是申请资源的最小单位，每个进程有`ownership`、`nice value`、`SELinux context`等诸多属性。Linux中，进程分为三类：用户进程（User Process）、守护进程（Daemon Process）、内核进程（Kernel Process）。

`用户进程（User Process）`是普通用户创建的、运行在用户态（User Space）的进程。除非进行了特殊的权限设置，否则用户进程没有其它用户文件的权限。

`守护进程（Daemon Process）`是在后台一直运行的进程，应当算是用户进程中的一种，但它不随着用户的退出而终止，而是一直在系统中运行，直到被人为关闭。

`内核进程（Kernel Process）`只在内核态运行，它也是常驻的进程，但内核进程拥有最高权限，可以访问所有的内核数据。

这是通常提到的三种划分方法，个人感觉这种划分方法在标准上不统一，严格说，按照进程的所属和权限，进程可以分为用户进程和内核进程，按照进程的形态，可以分为守护进程和非守护进程。用户进程既可以是守护进程也可以是非守护进程，内核进程都是守护进程。

## 进程的状态

进程是动态的，有多个状态。

正在被CPU执行的进程的状态是`Running`，没有被CPU执行的进程的状态是`Not Running`。

每个CPU核心同一时间只能执行一个进程，对一个CPU核心来说进程就是一组指令，CPU不停地吃进指令、执行执行，不能同时吃进两个指令（在CPU中使用流水线，可以一次给CPU塞进多个指令，但这些指令还是被一条一条执行的，不过CPU异常复杂，可能不能一言概之，但我们现在可以这样理解）。

将进程状态只分为`Running`和`Not Running`是远远不够的，进程需要有状态，其实是调度算法要求，调度算法的目的是让有限的CPU在兼顾效率和公平的情况下，处理数量远远超过CPU数量、且各种各样的进程。调度算法越复杂，进程的状态就越多。

现在，进程的状态主要有以下几种：

0. 刚创建，Born or forked
1. 可运行状态，Ready or Runnable
2. 运行状态，Running in User Or Running in Kernel
3. 阻塞状态，Blocked
4. 等待状态，Waiting
5. 睡眠状态，Sleeping
6. 睡眠状态又可以分为可中断（ Interruptable）和不可中断（ Uninterruptable ）两种状态
7. 睡觉状态还可以分为驻留在内存中和被交换到交换分区中两种状态
8. 僵尸状态、终止状态，Terminated or stopped

进程状态是`Runnable`，表示该进程万事具备，只缺CPU，可以随时被CPU执行。

进程状态是`Sleeping`，表示改进程需要的一些资源需要时间准备，调度器将原本被它占用的CPU调度给`Runable`状态的进程，直到进程需要的资源准备完成，重新进入Runnable状态。

用`ps -l`可以看到睡眠的进程停留在哪个内核函数上（`WCHAN`字段）（在加上参数`-e`可以显示所有进程）：

```bash
$ ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  7042  7040  0  80   0 - 28894 wait   pts/0    00:00:00 bash
0 R     0  7950  7042  0  80   0 - 34361 -      pts/0    00:00:00 ps
```

> WCHAN： name of the kernel function in which the process is sleeping, a "-" if the process is running, or a "*" if the process is multi-threaded and ps is not displaying threads.

处于睡眠状态的进程可以`可中断的`和`不可中断`的，两者的区别在于对信号或者事件的态度，前者在收到信号或者事件后，进程会跳出睡眠状态，后者不会，后者要等到系统调用完成后，才会感知到睡眠期间的发生事件。 `不可中断`的进程通常是在进行磁盘、网络I/O操作的进程，它们需要等到设备操作完成。

进程运行结束退出后，它在进程表中占用的位置，由它的父进程负责释放，在父进程还没有释放它占用的位置的时候，进程处于僵尸状态。

`ps -l`命令输出结果的第二列是进程所处的状态，ps一共展示以下几种状态：

	D    uninterruptible sleep (usually IO)
	R    running or runnable (on run queue)
	S    interruptible sleep (waiting for an event to complete)
	T    stopped by job control signal
	t    stopped by debugger during the tracing
	W    paging (not valid since the 2.6.xx kernel)
	X    dead (should never be seen)
	Z    defunct ("zombie") process, terminated but not reaped by its parent

在`man ps`中可以找到说明。

## 用户态与内核态

一个进程可以处于用户态和内核态，用户态和内核态其实是CPU的工作模式，CPU有相应的指令进行用户态和内核态的切换。CPU在内核态模式时，操作的是内核中的数据。用户态进程通过发起系统调用，进入到内核态。

系统调用可以理解为内核提供的功能，被调用的时候，CPU转为内核态，完成了相关操作后，重新切换为用户态，在用户态继续执行进程的后续指令。

`top`命令会打印出在过去2秒中（2s是默认值，可以用-d参数修改），每个CPU在每个模式中停留时间所占的比例：

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

在`man top`中可以找到这些说明。

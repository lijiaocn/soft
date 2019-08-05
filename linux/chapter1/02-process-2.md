<!-- toc -->
# 进程的状态

`ps -l`命令输出结果的第二列是进程所处的状态，ps一共展示以下几种状态：

	D    uninterruptible sleep (usually IO)
	R    running or runnable (on run queue)
	S    interruptible sleep (waiting for an event to complete)
	T    stopped by job control signal
	t    stopped by debugger during the tracing
	W    paging (not valid since the 2.6.xx kernel)
	X    dead (should never be seen)
	Z    defunct ("zombie") process, terminated but not reaped by its parent

在`man ps`中可以找到更详细说明。

## 进程状态介绍 

进程是动态的，是有状态的，正在被CPU执行的进程的状态是`Running`，没有被CPU执行的进程的状态是`Not Running`。在后面的火焰图章节中，正在运行的状态被称为`On-CPU`，没有在运行的状态被称为`Off-CPU`，一个进程要么在使用CPU，要么没有在使用CPU，在做性能分析的时候，这两段时间里发生的事情都需要关注。

![Thread state transition diagram](http://www.brendangregg.com/FlameGraphs/hotcoldfigure.png)

每个CPU核心同一时间只能执行一个进程，对一个CPU核心来说进程就是一组指令，CPU不停地吃进指令、执行执行，不能同时吃进两个指令（在CPU中使用流水线，可以一次给CPU塞进多个指令，但这些指令还是被一条一条执行的，不过CPU异常复杂，不能一言概之，我们现在可以这样理解）。

将进程状态分为`Running`和`Not Running`是远远不够的，进程的状态取决于调度算法，调度算法的目的是在兼顾效率和公平的情况下，让有限的CPU处理远超过CPU数量、各种各样的进程。调度算法越复杂，进程的状态就越多。

现在，进程的状态主要有以下几种：

	0. 刚创建，Born or forked
	1. 可运行状态，Ready or Runnable
	2. 运行状态，Running in User Or Running in Kernel
	3. 阻塞状态，Blocked
	4. 等待状态，Waiting
	5. 睡眠状态，Sleeping
	6. 睡眠状态又可以分为可中断（ Interruptable）和不可中断（ Uninterruptable ）两种状态
	7. 睡觉状态分为驻留在内存中和被交换到交换分区中两种状态
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

睡眠状态的进程分为`可中断的`和`不可中断`的，两者的区别在于对信号或者事件的态度，前者在收到信号或者事件后，进程会跳出睡眠状态，后者不会，后者要等到系统调用完成后，才会感知到睡眠期间的发生事件。 `不可中断`的进程通常是在进行磁盘、网络I/O操作的进程，它们需要等到设备操作完成。

进程运行结束退出后，在进程表中占用的位置由它的父进程负责释放，在父进程还没有释放它占用的位置的时候，进程处于僵尸状态。

## 查找父进程

用pstree可以直接展示出一个进程的父进程，以及父进程的父进程，直到1号继承，-a 表示输出命令行选项、 p 表 PID、s 表示指定进程的父进程

```sh
$ pstree -aps 3084
systemd,1
  └─dockerd,15006 -H fd://
      └─docker-containe,15024 --config /var/run/docker/containerd/containerd.toml
          └─docker-containe,3991 -namespace moby -workdir...
              └─app,4009
                  └─(app,3084)
```

pstree命令位于rpm包psmisc中：

```sh
yum install -y psmisc
```


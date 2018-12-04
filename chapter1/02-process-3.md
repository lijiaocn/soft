# 用户态与内核态

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

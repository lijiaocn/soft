# Perf 技术原理，Performance Counters 子系统的配套工具

2009 年的时候，一个名为 Performance Counters 的子系统被提交到 kernel，该子系统提供“计数”事件的功能，邮件 [Performance Counters for Linux][1] 中有阐述。Perf 是该系统的配套工具。

## 系统调用接口

Performance Counters 子系统中的统计数据通过系统调用 [perf_event_open][2] 读取。

```c
#include <linux/perf_event.h>
#include <linux/hw_breakpoint.h>

int perf_event_open(struct perf_event_attr *attr,
                    pid_t pid, int cpu, int group_fd,
                    unsigned long flags);
```

参数 pid 是目标进程号，既可以读取指定进程的事件，也可以读取所有进程的事件，详情见 linux 手册页：[perf_event_open - set up performance monitoring][2]。

## 权限控制

[Perf Events and tool security][3] 中介绍了设置 perf 命令权限的方法，读取内核中的统计数据，需要有响应的权限。单纯作为使用者可以不关心这个过程，发行版以及用 yum 等命令安装时，会完成相关设置。

## 参考

[1]: https://lwn.net/Articles/337493/ "Performance Counters for Linux"
[2]: http://man7.org/linux/man-pages/man2/perf_event_open.2.html "perf_event_open - set up performance monitoring"
[3]: https://www.kernel.org/doc/html/latest/admin-guide/perf-security.html "Perf Events and tool security"

<!-- toc -->
# Linux高性能网络IO处理

## 单进程模式：select、poll与epoll

select和poll都是非阻塞的，监视一组文件描述的读写状态，对满足条件的文件描述符进行操作，实现单线程处理多请求。select监控的文件描述符数量有限制，32位系统默认1024个，poll对select进行了改进，只受系统文件描述符数量的限制。

select和poll采用的都是轮询文件描述符列表的方式，效率较低，并且文件描述符要在用户空间和内核空间来回传送，增加了处理成本。

epoll在内核中用红黑树管理文件描述符集合，避免了文件描述符的传递，同时使用事件驱动机制，不需要轮询文件描述符集合。

## 多进程模式1：主进程+多子进程

主进程初始化套接字bind()+listen()、管理子进程的生命周期，子进程通过accpet()、epoll_wait()处理相同的套接字。accept和epoll存在“惊群”问题，accept在Linux2.6中解决了，epoll在Linux4.5中，通过EPOLLEXCLUSIVE解决。

**惊群**： 网络I/O事件发生时，同时有多个进程被唤醒，但最终只有一个进程响应事件，其它进程被不必要的唤醒。

## 多进程模式2：端口复用

Linux3.9提供了`SO_REUSEPORT`，支持多进程复用同一个端口，内核保证将到达该端口的请求均衡分发给监听的进程。同时内核确保只有进程被唤醒，不存在惊群的问题。

![nginx1.9.1端口复用](/img/linux/port-reuse.png)

## 绕过内核协议栈1：DPDK

DPDK直接在用户态轮询硬件网卡，绕过了内核协议栈，还通过huagepage、cpu绑定、内存对齐、流水线并发等多种机制，提高报文的处理效率：

![dpdk工作原理](/img/linux/dpdk.png)

## 绕过内核协议栈2：XDP

XDP（eXpress Data Path）是Linux内核（4.8以上）提供的高性能网络数据路径，在网络包进入内核协议栈之前进行处理。XDP是基于Linux内核的eBPF机制实现的：

![XDP工作原理](/img/linux/xdp.png)

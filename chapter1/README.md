<!-- toc -->
# Linux性能优化

这个章节中记录的Linux性能优化相关的知识，就是如何诊断性能瓶颈，如何优化。

## 方法论

RED方法：监控服务的请求数（Rate）、错误数（Errors）、响应时间（Duration）。Weave Cloud在监控微服务性能时提出的思路。

USE方法：监控系统资源的使用率（Utilization）、饱和度（Saturation）、错误数（Errors）。

![USE方法常见指标分类](/img/linux/use-metrics.png)

## 基准测试工具

![基准测试工具](/img/linux/benchmark-tool.png)

## 性能分析工具

![性能分析工具](/img/linux/analyst-tool.png)

## CPU分析思路

![CPU指标](/img/linux/cpu-metrics.png)

![CPU性能分析](/img/linux/cpu-analyst.png)

![CPU分析工具](/img/linux/cpu-tools.png)

## 内存分析思路

![内存性能指标](/img/linux/memory-metrics.png)

![内存性能分析](/img/linux/cpu-analyst.png)

![内存分析工具](/img/linux/memory-tools.png)

## IO分析思路

![文件系统IO指标](/img/linux/file-io-metrics.png)

![IO性能分析](/img/linux/io-analyst.png)

![IO分析工具](/img/linux/file-io-tools.png)


## 网络分析思路

![网络性能指标](/img/linux/net-metrics.png)

![网络性能分析](/img/linux/net-analyst.png)

![网络性能工具](/img/linux/net-tools.png)

## 参考资料

相当一部分内容来自极客时间出品的倪鹏飞专栏《Linux性能优化》，是这个[专栏](/)的学习笔记。

另一份资料是IBM红宝书[Linux性能调优指南](https://lihz1990.gitbooks.io/transoflptg/content/)。

此外，[The Linux Documentation Project](http://tldp.org/)是一个非常好的资料库。

将硬件中断的处理任务分配个多个CPU：[SMP affinity and proper interrupt handling in Linux](http://www.alexonlinux.com/smp-affinity-and-proper-interrupt-handling-in-linux)

[Hidden Costs of Memory Allocation](https://randomascii.wordpress.com/2014/12/10/hidden-costs-of-memory-allocation/)

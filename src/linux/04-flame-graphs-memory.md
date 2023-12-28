# 用内存火焰图分析内存泄露

内存泄露已经有几种分析工具，例如通过模拟运行进行分析的[Valgrind](http://valgrind.org/docs/manual/mc-manual.html)，可以统计内存堆（heap）使用情况的[libtcmalloc](http://goog-perftools.sourceforge.net/doc/heap_profiler.html)链接库。

相比已有的工具，内存火焰图是非侵入式的旁路分析方法，不干扰目标应用的运行。制作CPU火焰图时，采样时捕捉的是正在占用CPU的函数的调用栈，制作内存火焰图时，采样时捕捉的是正在进行内存申请/释放的函数的调用栈。

进程的内存空间如下图所示，都是通过malloc()、free()等几个函数申请或释放内存：

![进程内存空间图](http://www.brendangregg.com/FlameGraphs/memorytracing_1000.png)

以这些内存管理函数为目标进行采样，就可以知道哪些函数正在频繁的使用这些内存管理函数，例如下图是以`malloc`函数为目标进程采样的结果：

![内存火焰图](http://www.brendangregg.com/FlameGraphs/malloc_perl1.svg)

火焰图的横轴依旧是按字母顺序排列的函数，宽度代表函数在采样结果中出现的比例，纵轴依然是调用栈，上层函数被下层函数调用。但是，最顶层的函数的含义发生了变化，在CPU火焰图中，最顶层的函数是正在使用CPU的函数，在内存火焰图中，最顶层的函数是正在使用指定的内存管理函数的函数。

## malloc()等函数的采集

## brk()函数的采集

## mmap()函数的采集

## 缺页（Page Faults）采集


## 参考

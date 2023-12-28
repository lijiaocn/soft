# 火焰图介绍

火焰图是一个非常好的分析工具，可以直观地显示每个函数耗费的时间占比。

火焰图是性能优化领域大神[Brendan D. Gregg](http://www.brendangregg.com/)的杰作，他在[Flame Graphs](http://www.brendangregg.com/flamegraphs.html)一文里详细介绍了火焰图的用法。
并且贡献了一个火焰生成工具[FlameGraph](https://github.com/brendangregg/FlameGraph)，这个工具可以将调用栈采样数据转换成火焰图，[Perf Examples](http://www.brendangregg.com/perf.html#FlameGraphs)中提到了它的用法。采样数据也可以用其它工具采集，比如OpenResty的作者章宜春贡献的[stapxx](https://github.com/openresty/stapxx)。

Brendan D. Gregg的英文文章看起来吃力，可以参考阮一峰写的[如何读懂火焰图？](http://www.ruanyifeng.com/blog/2017/09/flame-graph.html)。

火焰图是这个样子的（它是一个动态图，右键查看图像）：

![火焰图](../img/linux/cpu-mysql-updated.svg)

横向是按照字母顺序排序的函数名，每个函数的宽度占比等于这个函数在采样中出现次数占比，也就等同于它耗费的时间比例。越宽的函数，耗费的的时间占比越大。

纵向是调用关系，上层的函数被下层的函数调用，最上层的函数是采样时正在占用CPU的函数。

如果在最上层存在很宽的函数，那么这个函数就是导致性能无法提升的罪魁祸首，需要重点分析优化。

火焰图的颜色没有特殊含义，只是为了方便查看，每个函数的颜色都是随机生成的，千万不要误以为和颜色有关，你应当关注的是函数的宽度。

任何性能工具生成的采用数据都可以用来生成火焰图：

	Linux: perf, eBPF, SystemTap, and ktap
	Solaris, illumos, FreeBSD: DTrace
	Mac OS X: DTrace and Instruments
	Windows: Xperf.exe

八卦一下：

Brendan D. Gregg在[文章](www.brendangregg.com/flamegraphs.html)中写道，他是在解决MySQL的性能问题时，产生了火焰图的创意。但是他采集了很多的文本数据，不方便查看，于是研究了一下怎样将采样数据可视化。他借鉴了Neelakanth and Roch的相关研究，用采样占比取代了时间占比，只选取了暖色调的颜色，生成了火焰一般的图形。这项工作是在2011年完成。

后来火焰图的创意被Google借鉴，植入了Chrome浏览器，用来分析网页性能，在开发者工具的性能选项卡中可以看到，不过chrome中的火焰图和这里的火焰图不一样，它的x轴是时间轴，并且是倒火焰形状，上层函数调用下层函数。

![Chrome中的火焰图](http://www.ruanyifeng.com/blogimg/asset/2017/bg2017092505.jpg)

在具体使用上，火焰图可以用来分析CPU占用情况、内存申请情况、未占用CPU时的情况，并且演化出“冷热（aHHot/Code）和“差异火焰图（Differential）”。


## 参考

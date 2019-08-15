# 冷热火焰图：将On-CPU火焰图与Off-CPU火焰图合并

把On-CPU火焰图和Off-CPU火焰图合并在一起或许更便于分析。


## 独立排列

这种方式其实就是放在一起而已，勉强算是一种融合方法吧。

![冷热火焰图组合-独立排列](/img/linux/cpu-mysql-filt-500.png)

## 共享横轴

这种方式是将火焰图的横轴轴对接起来共用，纵轴上依然是泾渭分明，有明确的边界：

![冷热火焰图组合-共享横轴](/img/linux/hotcoldthread-kernel.svg)

## 融合显示

这种方式是尽可能的On-CPU和Off-CPU糅合在一起：

![冷热火焰图组合-融合显示](/img/linux/eflame-blocking.png)



## 参考

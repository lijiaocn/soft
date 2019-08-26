<!-- toc -->
# Prometheus 的查询函数

Prometheus 的查询函数数量比较多，这里不罗列了，见 [Prometheus Functions][1]。

以后逐渐把不是特别好理解的函数的用法整理到这里。

## 区间数组

区间数组（range-vector）是过去一段时间内的多个数值，数值（scalar）。

### <aggregation>_over_time() -- 区间内聚合运算

这一组函数有多个，分别计算区间数组中的数据的平均值、最小值、最大值等：

形态：

```sh
avg_over_time(range-vector)
min_over_time(range-vector)
max_over_time(range-vector)
sum_over_time(range-vector)
count_over_time(range-vector)
quantile_over_time(scalar, range-vector)
stddev_over_time(range-vector)
stdvar_over_time(range-vector)
```

### holt_winters() -- 平滑数值

形态：holt_winters(v range-vector, sf scalar, tf scalar)

根据区间数组中的数据计算下一个平滑值，sf 是历史数值的权重，tf 是最新数值的权重，sf 和 tf 的取值范围为 [0,1]。

### increase() 

形态：increase(v range-vector)

计算区间数组中的增加值。

### idelta()

形态：idelta(v range-vector)

返回区间数组中最新的两个数值的差值。

### irate()、rate()

形态：irate(v range-vector)

用区间数组中最新的两个数值计算每秒变化。

形态：rate(v range-vector)

用区间数组中的数值计算每秒变化。

### resets()

形态：resets(v range-vector)

返回数组中被重置的计数器（counter）的数量。

### changes() -- 指定时间段内变化次数

形态：changes(v range-vector)

计算 range-vector（区间数组）中的数值变化的次数。

### delta()

形态：delta(v range-vector)

计算区间数组中第一个数值与最后一个数值的差值。

### deriv() -- 线性回归

形态：deriv(v range-vector)

### predict_linear()

形态：predict_linear(v range-vector, t scalar)

线性预测时长 t 之后的数值。


### 数组

数组（instant-vector）是同一时刻的多个数值，

### abs() -- 算绝对值

形态：abs(v instant-vector)

将数组中的数值转换成各自的绝对值。

### absent() -- 缺值判断

形态：absent(v instant-vector)

如果指标为空，结果为 1，否则结果为空。例如下图中指标查询结果为空，absent 的结果是 1：

[prometheus的absent函数用法](../img/prom/absent.png)

### ceil() -- 归整

形态：ceil(v instant-vector)

将数组中的数值转换成各自最接近的整数。

### floor() -- 向下归整

形态：floor(v instant-vector)

### clamp_max() -- 上限转换

形态：clamp_max(v instant-vector, max scalar)

将数组中超过上限的数值转换成上限值 max。

### clamp_min() -- 下限转换

形态：clamp_min(v instant-vector, min scalar)

将数组中低于下限的数值转换成下限值 min。


### exp()

形态：exp(v instant-vector)

### histogram_quantile() -- 分位数

形态：histogram_quantile(φ float, b instant-vector)

返回数组中的 φ 分位数 [0,1]。


### round()

形态：round(v instant-vector, to_nearest=1 scalar)

将数组中的数据向上归整，to_nearest 是归整步长。

The optional to_nearest argument allows specifying the nearest multiple to which the sample values should be rounded. This multiple may also be a fraction.

### scalar()

形态：scalar(v instant-vector)

将只有一个成员的数组转换成数值。

### sort()、sort_desc()

形态：sort(v instant-vector)、sort_desc(v instant-vector)

分别为升序、降序排列。

### sqrt()

形态：sqrt(v instant-vector)

计算平方根。

### label_join()

形态：label_join(v instant-vector, dst_label string, separator string, src_label_1 string, src_label_2 string, ...)

将多个 src_label_XX 拼接成一个新的 dst_label，用分隔符 separator 连接。

### label_replace()

形态：label_replace(v instant-vector, dst_label string, replacement string, src_label string, regex string)

用正则 regex 提取 src_label 中的字段，按照 replacement 形态拼接成 dst_label。

```sh
label_replace(up{job="api-server",service="a:c"}, "foo", "$1", "service", "(.*):.*")
```

### ln()

形态：ln(v instant-vector)

计算数组中的数值的对数。

### log2() 、log10()

形态：log2(v instant-vector)、log10(v instant-vector)

### vector()

形态：vector(s scalar)

将数值转换成数组。

## 时间换算

### day_of_month() -- 月中位置

形态：day_of_month(v=vector(time()) instant-vector)

返回宿主中的时间位于月中第几天。

### day_of_week() -- 周中位置

形态：day_of_week(v=vector(time()) instant-vector)

返回宿主中的时间位于周中第几天。

### days_in_month() -- 月份天数

形态：days_in_month(v=vector(time()) instant-vector)

返回宿主中时间所在月份的天数。

### year()

形态：year(v=vector(time()) instant-vector)

### month()

形态：month(v=vector(time()) instant-vector)

### hour()

形态：hour(v=vector(time()) instant-vector)

返回数组中时间位于当天的第几个小时。

### minute()

形态：minute(v=vector(time()) instant-vector)

### time()

形态：time()

返回当前的 UTC 时间（秒）

### timestamp()

形态：timestamp(v instant-vector)

返回数组中数值对应的时间戳。

## 参考

[1]: https://prometheus.io/docs/prometheus/latest/querying/functions/ "Prometheus Functions"

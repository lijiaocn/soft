<!-- toc -->
# Prometheus 指标查询结果运算

Prometheus 的查询语句支持运算，可以使用二元运算符（算术运算符、比较运算符、集合运算符）和聚合运算符直接操作查询出来的数据集，也可以使用 Prometheus 提供的查询函数进行更复杂的运算。

运算符操作的对象主要是 Instant vector、Scalar 类型。操作对象中存在 Instant vector 时，需要知道数组成员如何配对。

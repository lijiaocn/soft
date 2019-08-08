# Prometheus 查询结果二元运算

两组查询出来的数据集之间的运算。

## vector 的配对问题

操作数是两个 vector 时，vector 之间的配对关系可以是 1:1、1:N、N:1。

**1:1**，左边 vector 中数据 与 右边 vector 中拥有相同 label 的数据配对，可以用 ignoring 忽略部分 label，或者用 on 指定配对使用的 label，用法如下：

```sh
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>
```

**1:N，N:1**，group_left 表示右边 vector 中的一个成员会匹配左边的 vector 中的多个成员，group_right 反过来，左边 vector 中的一个成员会匹配右边的 vector 中的多个成员，group 指定的是 N。ignoring 和 on 分别用于删减用于匹配的 label 、指定用于匹配的 label：

```sh
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>

<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>
```

## vector 配对示例

假设 Prometheus 中存放下面两个指标数据：

```sh
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
```

1:1 配对，计算每个方法中 500 错误的占比，因为 method:http_requests:rate5m 没有名为 code 的 label，所以用 ignoring 忽略 code，只剩下 method label 用于配对 ：

```sh
method_code:http_errors:rate5m{code="500"} / ignoring(code) method:http_requests:rate5m
```

结果如下：

```sh
{method="get"}  0.04            //  24 / 600
{method="post"} 0.05            //   6 / 120
```

N:1 配对，计算每个方法中所有错误码的占比，多个错误代码对应一类请求：

```sh
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m
```

结果如下：

```sh
{method="get", code="500"}  0.04            //  24 / 600
{method="get", code="404"}  0.05            //  30 / 600
{method="post", code="500"} 0.05            //   6 / 120
{method="post", code="404"} 0.175           //  21 / 120
```

## 算术运算符

算术运算符支持：

	+、-、*、/（除法）、%（取模）、^（指数）。

## 比较运算符

比较运算符支持：

	==、!=、>、<、>=、<=

## 集合运算符

集合运算符支持：

	and（交集）、or（并集）、unless（差集）

集合运算符需要特别说明一下，vector1 and  vector2 的意思从 vector1 中取出满足 vetctor2 筛选条件的指标，例如下面的表达式：

```sh
http_server_requests_count{status="200"} and http_server_requests_count{method="POST",instance="10.12.3.5:8866"}

等同于：

http_server_requests_count{status="200",method="POST",instance="10.12.3.5:8866"}
```

vector1 or vector2 是取出 vector1 的所有成员 和 vector2 中不满足 vector1 的筛选条件的成员。

```sh
# 结果中包含所有满足 method="POST" 的数据，如果重复选择 or 之前的数据。
http_server_requests_count{status="200",instance="10.12.3.5:8866"} or http_server_requests_count{method="POST"}
```

vector1 unless vector2 取出不满足 vector2 筛选条件的所有 vector1 的成员：

```sh
http_server_requests_count{status="200",instance="10.12.3.5:8866"} unless http_server_requests_count{method="POST"}

等同于：

http_server_requests_count{status="200",instance="10.12.3.5:8866",method!="POST"}
```


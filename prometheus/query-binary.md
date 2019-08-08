<!-- toc -->
# Prometheus 查询结果的二元运算

两组查询出来的数据集之间的运算。

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
```

等同于：

```sh
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
```

等同于：

```sh
http_server_requests_count{status="200",instance="10.12.3.5:8866",method!="POST"}
```

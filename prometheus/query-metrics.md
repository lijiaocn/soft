# Prometheus 指标查询

Prometheus 的指标查询语句基本格式为：

	指标名称{ 标签名=<数值> }

以 http_server_requests_count 为例，指标上带有 label，通过 label 区分不同来源的数据。

查询 method 为 POST，status 为 200 的采集数据：

```sh
http_server_requests_count{method="POST",status="200"}
```

标签条件支持 `=`、`!=`、`=~`（正则匹配）。

默认查询的是当前时间的数据，如果要查询过去的数据，使用 offset，例如查询 5 分钟前的数据：

```sh
http_server_requests_count{method="POST",status="200"} offset 5m
```

要查询指标在某一区间的数值，使用 []，[] 中是从当前时间相对于相对于 offet向前推的时间段，例如查询 5 分钟前的 1 分钟区间里的数据：

```sh
http_server_requests_count{method="POST",status="200"}[1m] offset 5m
```

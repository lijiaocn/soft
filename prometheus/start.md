# Prometheus 安装启动

从 [prometheus download][1] 下载已经编译好的 prometheus 程序，该页面上还有 prometheus 提供的 exporter。

Prometheus 的命令行参数不是特别多，比较重要的有：

* --config.file              指定配置文件
* --log.level                日志级别
* --web.listen-address       监听地址
* --web.read-timeout         访问超时时间
* --web.enable-admin-api     启用 admin api
* --web.external-url         访问地址
* --query.max-concurrency    远程调用并发上限
* --storage.tsdb.path        指定数据文件存放目录
* --storage.tsdb.retention.time            设置数据保留期限，例如 15d
* --storage.remote.read-concurrent-limit   读取并发连接数上限

配置文件 prometheus.yml 中主要包含告警规则文件、静态配置的采集地址或者动态发现采集地址的方法。

## 参考

[1]: https://prometheus.io/download/ "prometheus download"

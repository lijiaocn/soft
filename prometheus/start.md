<!-- toc -->
# Prometheus 安装启动

从 [prometheus download][2] 下载已经编译好的 prometheus 程序，该页面上还有 prometheus 提供的 exporter。

## 命令行参数

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

## 配置文件格式

配置文件是 yaml 格式的，分为 global、alerting、rule_files 和 scrape_configs 四部分。


global 是全局参数配置，alerting 是告警服务地址，rule_files 是告警和记录规则，scrape_configs 是数据采集地址。其中 rule_files 和 scrape_configs 是重点，后者支持多种动态发现方式，更复杂一些。

```yaml
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
```

## 配置文件检查

可以用 promtool 检查配置文件是否正确：

```sh
$ ./promtool check config prometheus.yml 
Checking prometheus.yml
  SUCCESS: 0 rule files found
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://prometheus.io/download/ "prometheus download"

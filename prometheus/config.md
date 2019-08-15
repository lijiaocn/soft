# Prometheus 的配置文件

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

可以用 promtool 检查配置文件是否正确：

```sh
$ ./promtool check config prometheus.yml 
Checking prometheus.yml
  SUCCESS: 0 rule files found
```

## 参考

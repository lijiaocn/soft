# Prometheus 的 exporters

Prometheus 的负责收集指标数据、执行告警规则、提供指标数据查询功能，指标数据要么是目标系统实现了 Prometheus 样式的指标数据，要么通过各种 exporter 获取。[Third-party exporters][2] 中收录了很多 exporter，本手册逐渐收录工作中用到的一些 exporter。

* [blackbox exporter](./blackbox.md)：探测目标服务，支持 HTTP HTTPS DNS TCP ICMP

## 参考 

[1]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/08/03/prometheus-usage.html#_exporter "新型监控告警工具prometheus"
[2]: https://prometheus.io/docs/instrumenting/exporters/#third-party-exporters "Third-party exporters"

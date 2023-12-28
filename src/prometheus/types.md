<!-- toc -->
# Prometheus 的数据类型

Prometheus 存放的是每个指标的时间序列值，指标名由 Metric name 和 labels 组成 :

```sh
<metric name>{<label name>=<label value>, ...}
```

例如：

```sh
api_http_requests_total{method="POST", handler="/messages"}
```

## 指标类型

指标类型一共有 [四种][3]：

**Counter**：计数器，记录的是正向增长的累计值；

**Gauge**：  测量值，记录的是指标当前的状态数值；

**Histogram**： 直方图，就是统计学中的直方图，记录落在每个区间内的指标值的个数；

**Summary**：   分布图，就是统计学中的分布图，记录指标值的分位数。

## 生成代码

如果要生成 prometheus 样式的指标数据，可以用 prometheus 提供的 client sdk。go client 中的指标定义方法如下：

```go
scheduleAttempts = prometheus.NewCounterVec(
    prometheus.CounterOpts{
        Subsystem: SchedulerSubsystem,
        Name:      "schedule_attempts_total",
        Help:      "Number of attempts to schedule pods, by the result. 'unschedulable' means a pod could not be scheduled, while 'error' means an internal scheduler problem.",
    }, []string{"result"})
// PodScheduleSuccesses counts how many pods were scheduled.
PodScheduleSuccesses = scheduleAttempts.With(prometheus.Labels{"result": "scheduled"})

PreemptionVictims = prometheus.NewGauge(
    prometheus.GaugeOpts{
        Subsystem: SchedulerSubsystem,
        Name:      "pod_preemption_victims",
        Help:      "Number of selected preemption victims",
    })

BindingLatency = prometheus.NewHistogram(
    prometheus.HistogramOpts{
        Subsystem: SchedulerSubsystem,
        Name:      "binding_duration_seconds",
        Help:      "Binding latency in seconds",
        Buckets:   prometheus.ExponentialBuckets(0.001, 2, 15),
    },
)

SchedulingLatency = prometheus.NewSummaryVec(
    prometheus.SummaryOpts{
        Subsystem: SchedulerSubsystem,
        Name:      SchedulingLatencyName,
        Help:      "Scheduling latency in seconds split by sub-parts of the scheduling operation",
        // Make the sliding window of 5h.
        // TODO: The value for this should be based on some SLI definition (long term).
        MaxAge: 5 * time.Hour,
    },
    []string{OperationLabel},
)
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://prometheus.io/docs/concepts/data_model/ "DATA MODEL"
[3]: https://prometheus.io/docs/concepts/metric_types/ "METRIC TYPES"

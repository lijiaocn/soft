# Prometheus 的使用方法

[Prometheus][1] 是一个比较流行的告警监控系统，以前写过几篇笔记，这里更系统的整理一下。

这本手册的内容会根据实际工作的需要进行扩展，比较贴近实际。

手册目录在页面左侧，如果是在手机端查看，点击左上角的 <i class="fa fa-align-justify"></i> 符号。

**视频讲解（基于历史文章，最新的准备中）**：[Prometheus普罗米修斯监控入门](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1005950011&_trace_c_p_k2_=66a5b0594a3349fa815b4b135d6b2de6) 

## 历史文章收录

下面是 **以前的** 笔记，内容相对杂乱，不如当前手册条理、清晰，仅供参考：

* [《Prometheus的HTTP API的Go语言封装client_golang的使用》](https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2019/04/29/prometheus-go-client-usage.html)
* [《curl能访问的url，通过blackbox-expoeter进行探测时，返回404》](https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2018/12/03/prometheus-blackbox-exporter-return-404.html)
* [《使用Prometheus SDK输出Prometheus格式的Metrics》](https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/09/25/prometheus-client-usage.html)
* [《通过Prometheus查询计算Kubernetes集群中的容器CPU、内存使用率等指标》](https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2018/09/14/prometheus-compute-kubernetes-container-cpu-usage.html)
* [《新型监控告警工具prometheus（普罗米修斯）入门使用（附视频讲解）》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/08/03/prometheus-usage.html)
* [《Prometheus（普罗米修斯）使用过程中遇到的问题》](https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2018/08/03/prometheus-problem.html)

## 参考

[1]: https://prometheus.io/ "Prometheus"

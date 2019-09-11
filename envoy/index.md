# Envoy

**手册目录在页面左侧，如果是在手机端查看，点击左上角的 <i class="fa fa-align-justify"></i> 符号。**

初次了解 Envoy 时做过一些笔记，这里是再次学习时，对原先笔记的进一步整理。

这本手册的内容会根据实际工作的需要进行扩展，比较贴近实际。

**视频讲解**：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

## 环境和素材准备

```sh
docker pull envoyproxy/envoy:v1.11.0
git clone https://github.com/introclass/go-code-example.git
```

## 历史文章收录

下面是 **以前的** 笔记，内容相对杂乱，不如当前手册条理、清晰，仅供参考：
 
* [《Envoy Proxy使用介绍教程（一）：新型L3~L7层访问代理软件Envoy的使用》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/12/envoy-01-usage.html)
* [《Envoy Proxy使用介绍教程（二）：envoy源代码阅读、集成开发环境(IDE)》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/17/envoy-02-ide.html)
* [《Envoy Proxy使用介绍教程（三）：envoy设计思路、配置文件和功能特性概览》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/20/envoy-03-arch.html)
* [《Envoy Proxy使用介绍教程（四）：envoy源代码走读&启动过程分析》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/24/envoy-04-codes.html)
* [《Envoy Proxy使用介绍教程（五）：envoy的配置文件完全展开介绍》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/27/envoy-05-configfile.html)
* [《Envoy Proxy使用介绍教程（六）：envoy一些简单功能/基础配置的使用方法》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/28/envoy-06-features-1-basic.html)
* [《Envoy Proxy使用介绍教程（七）：envoy动态配置xDS的使用方法》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/29/envoy-07-features-2-dynamic-discovery.html)
* [《Envoy Proxy使用介绍教程（八）：envoy动态配置-聚合发现ADS的使用方法》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2019/01/07/envoy-08-features-3-dynamic-discovery-ads.html)
* [《Envoy Proxy使用介绍教程（九）：envoy的应用方法与使用约束》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2019/01/07/envoy-09-usage-rules.html)

## 参考

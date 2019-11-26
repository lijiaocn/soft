<!-- toc -->
# istio 的四大功能和基本概念

## istio 的四大功能

istio 的功能分为四类：

* 流量管理，[Traffic Management](https://istio.io/docs/concepts/traffic-management/)
* 认证加密，[Security](https://istio.io/docs/concepts/security/)
* 访问微调，[Policies](https://istio.io/docs/concepts/policies/)
* 全局监控，[Observability](https://istio.io/docs/concepts/observability/)

![istio-arch](https://www.lijiaocn.com/img/article/istio-arch.svg)

## istio 的基本概念

istio 的规则配置主要围绕下面的概念进行：

* [VirtualService](./vsvc.md)， 转发规则
* [DestinationRule](./dstrule.md)，均衡策略
* [Gateways](./gateway.md)，对外服务
* [ServiceEntry](./entry.md)，封装外部服务
* [Engress Control](./egress.md), 一种管控外出请求的方法

如果非要类比的话：

* VirtualService 相当于 nginx 中的 Server
* DestinationRule 相当于 nginx 中的 upstream 
* Gateway 和 ServiceEntry 相当于 kubernetes 中的 ingress 和 endpoints

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

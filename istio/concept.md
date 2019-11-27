<!-- toc -->
# istio 的四大功能和基本概念

## istio 的四大功能

istio 的功能分为四类：

* 流量管理，[Traffic Management](https://istio.io/docs/concepts/traffic-management/)
* 认证加密，[Security](https://istio.io/docs/concepts/security/)
* 访问策略，[Policies](https://istio.io/docs/concepts/policies/)
* 全局监控，[Observability](https://istio.io/docs/concepts/observability/)

![istio-arch](https://www.lijiaocn.com/img/article/istio-arch.svg)

## istio 的基本概念

istio 的规则配置主要围绕下面的概念进行：

* [VirtualService](./vsvc.md)， 转发规则
* [DestinationRule](./dstrule.md)，均衡策略
* [Gateway](./gateway.md)，对外服务
* [ServiceEntry](./entry.md)，封装外部服务
* [Engress Control](./egress.md), 一种管控外出请求的方法

如果非要类比的话：

* VirtualService 相当于 nginx 中的 Server
* DestinationRule 相当于 nginx 中的 upstream 
* Gateway 和 ServiceEntry 相当于 kubernetes 中的 ingress 和 endpoints

## istio 的其它概念

istio 定了了大量的 CRD，除了上面的 VirtualService、DestinationRule、Gateway、ServiceEntry。初次之外，还有：

[指标的采集](./metrics.md) 中用到的：

	instance（指标定义）
	handler（收集器）
	rule（采集动作）

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

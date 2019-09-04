<!-- toc -->
# Istio 的基本概念

Istio 的规则配置围绕三个基本概念进行：

* [Virtual services](istio/vsvc.md)， 转发规则
* [Destination rules](istio/dstrule.md)，均衡策略
* [Gateways](istio/gateway.md)，对外服务

[Virtual services](istio/vsvc.md) 是核心概念，[Destination rules](istio/dstrule.md) 和 [Gateways](istio/gateway.md) 都在 VirtualService 中引用。

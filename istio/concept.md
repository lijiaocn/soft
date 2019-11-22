<!-- toc -->
# Istio 的基本概念

Istio 的规则配置主要围绕下面的概念进行：

* [VirtualService](./vsvc.md)， 转发规则
* [DestinationRule](./dstrule.md)，均衡策略
* [Gateways](./gateway.md)，对外服务
* [ServiceEntry](./entry.md)，封装外部服务
* [Engress Control](./egress.md), 一种管控外出请求的方法

如果非要类比的话：

* Virtual Services 相当于 nginx 中的 Server
* Destination Rules 相当于 nginx 中的 upstream 
* Gateways 和 Service Entry 是 istio 特有的概念

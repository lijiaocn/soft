<!-- toc -->
# Istio 的流出流量管控

Engress Control 不是 istio 的基本概念，只是一种用法。istio 不仅能够管控从外部流入的流量（通过 [Service Entry](./entry.md)，也可以管控流出的流量。

流出流量的管控稍微复杂一些，同时用到 [VirtualService](./vsvc.md)、[ServiceEntry](./entry.md) 和 [Gateway](./gateway.md)，因此单独列一篇。

基本思路是：

将外部服务封装成 ServiceEntry，然后创建一个 Gateway，指示 engress envoy 监听，最后创建一个对应的 VirtualService，将网格内对外部服务请求的转发到 engress envoy，将 engress envoy 收到的请求转发到 ServiceEntry（即外部服务）。

## 将外部服务封装成 ServiceEntry

下例中，封装后的外部服务在网格内的名称是 httpbin.com（更多封装方法见 [ServiceEntry](./entry.md)）：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-httpbin
  namespace : egress
spec:
  hosts:
  - httpbin.com
  exportTo:
  - "."
  location: MESH_EXTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
```

## 创建接收外出流量的 Gateway

创建 Gateway，选定带有标签 "istio: egressgateway"  的边界 envoy，指示它监听 80 端口，准备接收发送到外部的流量：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
 name: istio-egressgateway
 namespace: istio-system
spec:
 selector:
   istio: egressgateway
 servers:
 - port:
     number: 80
     name: http
     protocol: HTTP
   hosts:
   - "*"
```

## 创建包含转发规则的 VirtualService

VirtualService 的 hosts 是 ServiceEntry 的 hosts ，同时绑定上面的创建 Gateway，设置网格内流量的转发规则，和 Gateway 收到的流量的转发规则：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: gateway-routing
  namespace: egress
spec:
  hosts:
  - httpbin.com
  exportTo:
  - "*"
  gateways:
  - mesh
  - istio-egressgateway
  http:
  - match:
    - port: 80
      gateways:
      - mesh
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
  - match:
    - port: 80
      gateways:
      - istio-egressgateway
    route:
    - destination:
        host: httpbin.com
```

VirtualService 中配置了两个转发规则：

* 网格内的请求，即从 mesh 到来的流量转发给边界 envoy 在 kubernetes 中的服务域名 istio-egressgateway.istio-system.svc.cluster.local，也就是发送到边界了 envoy，边界 envoy 已经在 Gateway 的指示监听 80 端口。

* 通过 Gateway 的 80 端口到来的流量，也就是边界 envoy 接收到的流量（包含从网格内部转来的流量和从外部发送到边界 envoy 的流量），统统转发给外部服务 httpbin.com。

通过这种方式，将网格内部发起的对外部服务请求，转发到指定的边界 envoy 送出，实现了统一的出口。

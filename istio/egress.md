<!-- toc -->
# Istio 的流出流量管控

Engress Control 不是 istio 的基本概念，只是一种用法，istio 不仅能够管控从外部流入的流量（通过 [Service Entry](./entry.md)，也可以管控流出的流量。

流出流量的管控稍微复杂一些，同时用到 VirtualService、ServiceEntry 和 Gateway，单独列一篇。

基本思路是：将外部服务封装成 ServiceEntry，然后创建一个代理该 ServiceEntry 的 VirtualService，将外出的流量转发到负责外出流量的边界 envoy。

## 将外部服务封装成 ServiceEntry

封装后的服务名就是 httpbin.com，更多封装方法见 [ServiceEntry](./entry.md)：

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

## 创建转发外出流量的 Gateway

带有标签 "istio: egressgateway"  的边界 envoy 将监听 80 端口，接收要通过它转发到外部的流量：

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

VirtualService 的 hosts 与 ServiceEntry 的 hosts 相同，同时绑定上面的 Gateway，destination 为边界 envoy ：

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

网格内的请求，即从 mesh 到来的流量转发给边界 envoy 在 kubernetes 中的服务域名 istio-egressgateway.istio-system.svc.cluster.local，也就是发送到边界 envoy。通过 Gateway 的 80 端口到来的流量，也就是边界 envoy 接收到流量，转发给封装的外部服务 httpbin.com。

通过这种方式，将网格内部发起的对外部服务请求，转发到指定的边界 envoy 送出。

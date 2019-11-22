<!-- toc -->
# istio 的流出流量管控

Engress Control 不是 istio 的基本概念，是一种用法。

istio 不仅能够管控从外部流入的流量（通过 [Service Entry](./entry.md)，也可以管控流出的流量。流出流量的管控稍微复杂一些，同时用到 [VirtualService](./vsvc.md)、[ServiceEntry](./entry.md) 和 [Gateway](./gateway.md)。

基本思路：

* 将外部的服务封装成一个 ServiceEntry
* 创建一个 Gateway，让 engress envoy 启动一个监听端口
* 创建一个对应的 VirtualService，将网格内对外部服务请求的转发到 engress envoy，
* 将 engress envoy 收到的请求转发到 ServiceEntry（即外部服务）

## 将外部服务封装成 ServiceEntry

封装后的外部服务在网格内的名称是 `httpbin.com`，后面的 VirtualService 会指向这个 host：

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

外部服务的封装方法有多中，见 [ServiceEntry](./entry.md)）。

## 创建接收外出流量的 Gateway

创建 Gateway，指示边界 envoy 监听 80 端口，后面的 VirtualService 会引用这个名为 ` istio-egressgatewa` 的 gateway。这里选定带有 "istio: egressgateway" 标签的边界 envoy，与接收外部请求的 envoy 区分开：


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

创建 hosts 为 httpbin.com 的 VirtualService，设置网格内发起的（mesh）和 istio-egressgateway 收到的到 httpbin.com 的请求的转发规则：


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

http 中的 两个 match 对应两个转发规则，针对都是 host 为 httpbin.com 的请求：

* 第一个规则：网格内发起的请求，转发到 istio-egressgateway.istio-system.svc.cluster.local，这个域名是 istio-egressgateway 选择的边界 envoy 在 kubernetes 中的地址，发送到这个地址就是发送到 istio-egressgateway。

* 第二个规则：istio-egressgateway 的 80 端口收到的请求 ，即边界 envoy 接收到的请求（包含第一个规则设置的从网格内部转来的流量和从外部发送到边界 envoy 的流量），统统转发给外部服务 httpbin.com。

在上面两个规则的作用下，从网格内（即 pod 中）发起的请求先转发给边界 envoy，再由边界 envoy 将其转发给外部的 httpbin.com，实现了对 `外出流量` 的流量集中管控，边界 envoy 就是外部服务的访问代理。

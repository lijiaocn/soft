<!-- toc -->
# Gateways

[Gateways][1] 用来管理南北向流量，也就是从外部流入网格，和从网格流出到外部的流量，gateway 中的配置作用于网格边界的 envoy ，处理流入流量的是 ingress gateway，处理流出流量的是 egress gateway。

![istio的请求流向](../img/envoy/ingress-egress.svg)

Gateway 详细内容见 [Envoy Gateway Detail][2]。

## 放行外部流量

下面的配置将被转换成边界 envoy （这里是带有 app: my-gateway-controller 的 envoy pod）中的配置，允许 host 为 ext-host、协议为 https 的外部流量进入网格：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: ext-host-gwy
spec:
  selector: 
    app: my-gateway-controller
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - ext-host
    tls:
      mode: SIMPLE
      serverCertificate: /tmp/tls.crt
      privateKey: /tmp/tls.key
```

selector 是筛选目标的 envoy pod，上面的配置效果为：带有 `app: my-gateway-controller` 标签的 envoy pod 监听 443端口，接收 host 为 ext-host 的请求。

[Envoy Gateway Detail][2] 中给出了一个更复杂的示例：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: my-gateway
  namespace: some-config-namespace
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - uk.bookinfo.com
    - eu.bookinfo.com
    tls:
      httpsRedirect: true # sends 301 redirect for http requests
  - port:
      number: 443
      name: https-443
      protocol: HTTPS
    hosts:
    - uk.bookinfo.com
    - eu.bookinfo.com
    tls:
      mode: SIMPLE # enables HTTPS on this port
      serverCertificate: /etc/certs/servercert.pem
      privateKey: /etc/certs/privatekey.pem
  - port:
      number: 9443
      name: https-9443
      protocol: HTTPS
    hosts:
    - "bookinfo-namespace/*.bookinfo.com"
    tls:
      mode: SIMPLE # enables HTTPS on this port
      credentialName: bookinfo-secret # fetches certs from Kubernetes secret
  - port:
      number: 9080
      name: http-wildcard
      protocol: HTTP
    hosts:
    - "*"
  - port:
      number: 2379 # to expose internal service via external port 2379
      name: mongo
      protocol: MONGO
    hosts:
    - "*"
```

## 绑定 VirtualService

Gateway 指示边界 envoy 监听指定的端口、允许特定的请求，但是没有告诉边界 envoy 要如何转发这些从外部进来的请求。转发规则在 [Virtual services](./vsvc.md) 中定义，要把从 gateway 进来的流量绑定到包含 route 的 VirtualService。

下面的配置将从 some-config-namespace/my-gateway 进来的、端口为 27017 的流量转发到 mongo.prod.svc.cluster.local：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo-Mongo
  namespace: bookinfo-namespace
spec:
  hosts:
  - mongosvr.prod.svc.cluster.local # name of internal Mongo service
  gateways:
  - some-config-namespace/my-gateway # can omit the namespace if gateway is in same
                                       namespace as virtual service.
  tcp:
  - match:
    - port: 27017
    route:
    - destination:
        host: mongo.prod.svc.cluster.local
        port:
          number: 5555
```

多个 Gateways 可以绑定到同一个 VirtualService，下面这个 VirtualService 绑定了 gateway 来的流量和网格内部的流量 ：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo-rule
  namespace: bookinfo-namespace
spec:
  hosts:
  - reviews.prod.svc.cluster.local
  - uk.bookinfo.com
  - eu.bookinfo.com
  gateways:
  - some-config-namespace/my-gateway
  - mesh # applies to all the sidecars in the mesh
  http:
  - match:
    - headers:
        cookie:
          exact: "user=dev-123"
    route:
    - destination:
        port:
          number: 7777
        host: reviews.qa.svc.cluster.local
  - match:
    - uri:
        prefix: /reviews/
    route:
    - destination:
        port:
          number: 9080 # can be omitted if it's the only port for reviews
        host: reviews.prod.svc.cluster.local
      weight: 80
    - destination:
        host: reviews.qa.svc.cluster.local
      weight: 20
```

## 限制可以绑定的 VirtualService

如果不希望 Gateway 被任意的 VirtualService 绑定，可以在 Gateway 的 hosts 中限制绑定范围：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: my-gateway
  namespace: some-config-namespace
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "ns1/*"
    - "ns2/foo.bar.com"
```

`ns1/*`：可以与 ns1 中的所有 VirtualService 绑定。

`ns2/foo.bar.com` 只能与 ns2 中包含 foo.bar.com 的 VirtualService 绑定。

## 参考

[1]: https://istio.io/docs/concepts/traffic-management/#gateways  "Gateways"
[2]: https://istio.io/docs/reference/config/networking/v1alpha3/gateway/ "Envoy Gateway Detail"

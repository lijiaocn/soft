<!-- toc -->
# istio 的转发功能使用方法

示例拆解中的 [Bookinfo Application](./bookinfo.md) 包含了转发功能，这里简单复述下。

转发策略用 [VirtualService](./vsvc.md) 描述，在 VirtualService 中配置 hostname、uri 和目标服务的对应关系，目标服务是域名或 IP，同一个 namespace 内的服务可以只用服务名称。

需要注意匹配条件中的 hosts，可以是应用的对外域名，实现域名到集群内服务的映射，也可以是同一个服务，实现服务到自身的映射。

## 外部域名到集群内服务的映射

外部域名到集群内服务的映射：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  - "bookinfo.xxx.com"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

## 集群内 Service 到自身的映射

集群内 Service 到自身的映射，下面的 hosts 和 host 的值都是 reviews：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
```

这样可以为到达 reviews 的请求设置策略，譬如上面的 v1 来自于 reivews 服务的 Destination：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
```

## VirtualService 支持的匹配规则

VirtualService 支持以下匹配规则：

* [HTTPMatchRequest][4]
* [TLSMatchAttributes][2]
* [L4MatchAttributes][3]

譬如匹配 headers：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

[2]: https://istio.io/docs/reference/config/networking/virtual-service/#TLSMatchAttributes "TLSMatchAttributes"
[3]: https://istio.io/docs/reference/config/networking/virtual-service/#L4MatchAttributes "L4MatchAttributes"
[4]: https://istio.io/docs/reference/config/networking/virtual-service/#HTTPMatchRequest "HTTPMatchRequest"

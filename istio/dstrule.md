<!-- toc -->
# Destination rules

[Destination Rule][3] 可以理解为负载均衡策略，它作用于 VirtualService 中的 destination，在 route 之后起作用，详细介绍见 [Destination Rule][3]。

## 将 Pod 按照 label 分组

下面是一个 DestinationRule，它作用于服务 my-svc，将这个服务对应的 pod 按照 label 再次分为 三组：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3
```

host 是 Destination Rule 作用的 kubernetes 中的 service。

subsets 是同时定义的多个不同策略，subsets 中的字段会覆盖前面出现过的字段，subsets 中不存在的字段使用前面出现的字段的值，即 subsets 外部定义的是配置字段的默认值。

subsets 中 labels 的作用是筛选 pod，上面的配置将 `my-svc` 服务的 pod 再次按照 label 分成了 v1、v2、v3 三组，从而可以为每组单独设置负载均衡、连接池策略等。

## 将请求按照端口分组

也可以按端口分别设置策略：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: bookinfo-ratings-port
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy: # Apply to all ports
    portLevelSettings:
    - port:
        number: 80
      loadBalancer:
        simple: LEAST_CONN
    - port:
        number: 9080
      loadBalancer:
        simple: ROUND_ROBIN
```

Destination Rule 中可以设置连接池、探活等策略，详情见 [Destination Rule][3]。

## 在 VirtualService 中使用 Destination Rule 

Destination Rule 中配置的 subsets 在 VirtualService 中引用，在 VirtualService 中指定 destination 时可以指定 subset，如下：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews-route
  namespace: foo
spec:
  hosts:
  - reviews # interpreted as reviews.foo.svc.cluster.local
  http:
  - match:
    - uri:
        prefix: "/wpcatalog"
    - uri:
        prefix: "/consumercatalog"
    rewrite:
      uri: "/newcatalog"
    route:
    - destination:
        host: reviews # interpreted as reviews.foo.svc.cluster.local
        subset: v2
  - route:
    - destination:
        host: reviews # interpreted as reviews.foo.svc.cluster.local
        subset: v1
```

## 参考

[1]: https://istio.io/docs/concepts/traffic-management/ "Traffic routing and configuration"
[2]: https://istio.io/docs/concepts/traffic-management/#virtual-services "Virtual services"
[3]: https://istio.io/docs/reference/config/networking/v1alpha3/destination-rule/ "Destination Rule"
[4]: https://istio.io/docs/concepts/traffic-management/#gateways "Gateways"

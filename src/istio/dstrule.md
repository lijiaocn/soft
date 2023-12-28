<!-- toc -->
# istio 的基本概念： DestinationRule

[DestinationRule][3] 是转发策略，在 route 规则之后起作用，相当于 nginx 中的负载均衡策略。它作用于 VirtualService 的 destination 中的同名的 host。

## 将 Pod 按照 label 分组，分别设置转发策略

下面的 DestinationRule 作用于 my-svc，影响到 my-svc 的请求的分配策略。

设置了全局的转发策略（subsets 以外的配置），同时将 my-svc 的 pod 按照 label 分成了三组，单独设置了每组的转发策略（subnets 内的配置）：

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

* host 指定作用的服务，与 VirtualService 的 Destination 中的 host 配对
* subsets 中的的转发策略覆前面 subsets 外的同名配置
* subsets 中没有的配置，使用 subsets 外的配置

subsets 中 labels 的作用是筛选 pod，上面的配置将 `my-svc` 的 pod 按照 label 分成了 v1、v2、v3 三组，为每组单独设置转发策略。

## 将请求按照端口分组，分别设置转发策略

除了按照 pod 分组，还可以按目的端口分组，为每个端口设置转发策略：

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

Destination Rule 的详情见 [Destination Rule][3]。

## 在 VirtualService 中使用 DestinationRule 

在 VirtualService 中指定 destination 时，引用 destination 中的 subset，如下：

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

<!-- toc -->
# DestinationRule

[Destination Rule][3] 是转发策略，它作用于 VirtualService 中的 destination 中的 host，也就是 kubernetes 中的服务。Destination Rule 在 route 规则之后起作用，详细介绍见 [Destination Rule][3]。

## 将 Pod 按照 label 分组，分别设置转发策略

下面是一个 DestinationRule，它作用于服务 my-svc，并且将这个服务包含的 pod 按照 label 分为三组：

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

subsets 中是多个转发策略，里面的配置会覆盖前面 spec 中出现过的同名配置，subsets 中不存在的配置使用前面出现的配置的值，即 subsets 外部定义的是默认配置。

subsets 中 labels 的作用是筛选 pod，上面将 `my-svc` 服务中的 pod 按照 label 分成了 v1、v2、v3 三组，为每组单独设置转发策略。

## 将请求按照端口分组，分别设置转发策略

还可以按端口分别设置转发策略：

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

DestinationRule 中配置的 subsets 在 VirtualService 中引用，在 VirtualService 中指定 destination 时可以指定 subset，如下：

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

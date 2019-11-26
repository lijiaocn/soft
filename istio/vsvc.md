<!-- toc -->
# istio 的基本概念：VirtualService

[VirtualService][2] 是 istio 的核心概念，它包含一组 route ，定义了请求转发规则，参考 [VirtualService][5]。 VirtualService 的配置格式如下：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-namespace
spec:
  hosts:
  - my-namespace.com
  http:
  - match:
    - uri:
        prefix: /svc-1
    route:
    - destination:
        host: svc-1.my-namespace.svc.cluster.local
  - match:
    - uri:
        prefix: /svc-2
    route:
    - destination:
        host: svc-2.my-namespace.svc.cluster.local
```

`hosts` 是域名匹配规则，限定了 virtualservice 的作用范围，即 virtualservice 中的规则只影响与 hosts 匹配的请求。hosts 匹配规则可以使用通配符。

`host`（注意和 hosts 字段区分开）是转发目标，符号要求的请求被转发给到 host 中配置的域名。域名既可以是 kubernetes 中的域名（可以跨 namespace），也可以是外部服务的域名。

VirtualService 还可以绑定 gateway（即只处理特定 gateway 的流量），它联结了 [DestinationRule](./dstrule.md)、[Gateways](./gateway.md) 和 [ServiceEntry](./entry.md)，后面章节中会频繁用到 VirtualService。

## 参考

[1]: https://istio.io/docs/concepts/traffic-management/ "Traffic routing and configuration"
[2]: https://istio.io/docs/concepts/traffic-management/#virtual-services "Virtual services"
[3]: https://istio.io/docs/reference/config/networking/v1alpha3/destination-rule/ "Destination Rule"
[4]: https://istio.io/docs/concepts/traffic-management/#gateways "Gateways"
[5]: https://istio.io/docs/reference/config/networking/v1alpha3/virtual-service/ "Envoy VirtualService Detail"

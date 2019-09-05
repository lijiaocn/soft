<!-- toc -->
# Virtual services

[Virtual services][2] 是 istio 的核心概念，它是一组 route，定义了请求转发的规则，详细内容见 [Virtual services][5]。 VirtualService 的配置格式如下：

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

`hosts` 是域名匹配规则，可以使用通配符。

`host` 是目标服务，即转发到的服务在 kubernetes 中的域名，可以转发给任意 namespace 中的 service。

VirtualService 是 istio 核心概念，联结了 [DestinationRule](./dstrule.md)、[Gateways](./gateway.md) 和 [ServiceEntry](./entry.md)，在后面章节说明。

## 参考

[1]: https://istio.io/docs/concepts/traffic-management/ "Traffic routing and configuration"
[2]: https://istio.io/docs/concepts/traffic-management/#virtual-services "Virtual services"
[3]: https://istio.io/docs/reference/config/networking/v1alpha3/destination-rule/ "Destination Rule"
[4]: https://istio.io/docs/concepts/traffic-management/#gateways "Gateways"
[5]: https://istio.io/docs/reference/config/networking/v1alpha3/virtual-service/ "Envoy VirtualService Detail"

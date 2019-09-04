# Virtual services

[Virtual services][2] 是 istio 的核心概念，它是一组 route，将满足特定条件的请求转发到指定的 service。详细内容见 [Virtual services][5]。


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

`hosts` 是请求的域名，可以使用通配符。

`host` 是转发到的服务在 kubernetes 中的域名，可以转发给任意 namespace 中的 service。

## 参考

[1]: https://istio.io/docs/concepts/traffic-management/ "Traffic routing and configuration"
[2]: https://istio.io/docs/concepts/traffic-management/#virtual-services "Virtual services"
[3]: https://istio.io/docs/reference/config/networking/v1alpha3/destination-rule/ "Destination Rule"
[4]: https://istio.io/docs/concepts/traffic-management/#gateways "Gateways"
[5]: https://istio.io/docs/reference/config/networking/v1alpha3/virtual-service/ "Envoy VirtualService Detail"

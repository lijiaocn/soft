<!-- toc -->
# istio 的访问策略

istio 提供了限速、按 header 路由等访问控制策略。

## istio 限速策略

istio 的 [限速功能][2] 用起来相当繁琐，用到 handler、instance、QuotaSpec、QuotaSpecBinding 和 rule 总共 5 个 CRD。（这么繁琐，想吐槽... 2019-11-27 18:36:29）

```sh
$ kubectl apply -f  samples/bookinfo/policy/mixer-rule-productpage-ratelimit.yaml
```

instance 定义了度量的维度，后面的 handler 会使用这些维度属性：

```yaml
apiVersion: config.istio.io/v1alpha2
kind: instance
metadata:
  name: requestcountquota
  namespace: istio-system
spec:
  compiledTemplate: quota
  params:
    dimensions:
      source: request.headers["x-forwarded-for"] | "unknown"
      destination: destination.labels["app"] | destination.service.name | "unknown"
      destinationVersion: destination.labels["version"] | "unknown"
```

handler 定义了基于内存的请求配额管理策略（memquota，众多配额管理器中的一种），设置了限速规则：

```yaml
apiVersion: config.istio.io/v1alpha2
kind: handler
metadata:
  name: quotahandler
  namespace: istio-system
spec:
  compiledAdapter: memquota
  params:
    quotas:
    - name: requestcountquota.instance.istio-system
      maxAmount: 500
      validDuration: 1s
      # The first matching override is applied.
      # A requestcount instance is checked against override dimensions.
      overrides:
      # The following override applies to 'reviews' regardless
      # of the source.
      - dimensions:
          destination: reviews
        maxAmount: 1
        validDuration: 5s
      # The following override applies to 'productpage' when
      # the source is a specific ip address.
      - dimensions:
          destination: productpage
          source: "10.28.11.20"
        maxAmount: 500
        validDuration: 1s
      # The following override applies to 'productpage' regardless
      # of the source.
      - dimensions:
          destination: productpage
        maxAmount: 2
        validDuration: 5s
```

rule 设置了限速的范围，对符合条件的请求启用限速：

```yaml
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: quota
  namespace: istio-system
spec:
  # quota only applies if you are not logged in.
  # match: match(request.headers["cookie"], "user=*") == false
  actions:
  - handler: quotahandler
    instances:
    - requestcountquota
```

QuotaSpec 是分配的限速配额，它将被绑定到要被限速的服务：

```yaml
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpec
metadata:
  name: request-count
  namespace: istio-system
spec:
  rules:
  - quotas:
    - charge: 1
      quota: requestcountquota
```

QuotaSpecBinding 将限速配额与目标服务绑定：

```yaml
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpecBinding
metadata:
  name: request-count
  namespace: istio-system
spec:
  quotaSpecs:
  - name: request-count
    namespace: istio-system
  services:
  - name: productpage
    namespace: default
    #  - service: '*'  # Uncomment this to bind *all* services to request-count
```

这些配置的最终效果是：

```sh
productpage 的访问频率限制为 5 秒 2 次，如源头 IP 是 10.28.11.20，访问限速为 500次/s。

reviews 的访问频率限制为 5 秒 1 次。

其它服务的访问频率限制为 500 次/s
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://istio.io/docs/tasks/policy-enforcement/rate-limiting/ "Enabling Rate Limits"

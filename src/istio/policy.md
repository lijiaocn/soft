<!-- toc -->
# istio 的访问策略

istio 提供了限速、按 header 路由等访问控制策略。

## istio 黑白名单

istio 基于 mixer 获取的属性设置黑白名单，黑白名单也是通过 instance、handler 和 rule 设置。

### 按标签设置黑白名单

下面三条规则禁止 productpage 访问 http-record v2：

```yaml
apiVersion: "config.istio.io/v1alpha2"
kind: handler
metadata:
  name: denyproductpagehandler
spec:
  compiledAdapter: denier
  params:
    status:
      code: 7
      message: Not allowed
---
apiVersion: "config.istio.io/v1alpha2"
kind: instance
metadata:
  name: denyproductpagerequest
spec:
  compiledTemplate: checknothing
---
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: denyproductpage
spec:
  match: destination.labels["app"] == "http-record" && destination.labels["version"] == "v2" && source.labels["app"]=="productpage"
  actions:
  - handler: denyproductpagehandler
    instances: [ denyproductpagerequest ]
```

### 按标签禁止访问

http-record v1 和 http-record v2 的 ip 地址如下：

```sh
$ kubectl get pod --show-labels |grep http-record
http-record-v1-5f9c95b7cf-t9tzx   2/2     Running   0  5d23h   172.17.0.16   app=http-record,pod-template-hash=5f9c95b7cf,security.istio.io/tlsMode=istio,version=v1
http-record-v2-d697c886-lqpsw     2/2     Running   0  5d23h   172.17.0.4    app=http-record,pod-template-hash=d697c886,security.istio.io/tlsMode=istio,version=v2
```

在 productpage 中访问 http-record v1：

```sh
$ curl 172.17.0.16:8080
{
    "RemoteAddr": "127.0.0.1:52814",
    "Method": "GET",
    "Host": "172.17.0.16:8080",
    "RequestURI": "/",
    "Header": {
    ...省略...
```

在 productpage 中访问 http-record v2：

```sh
$ curl -v 172.17.0.4:8080
* Rebuilt URL to: 172.17.0.4:8080/
*   Trying 172.17.0.4...
* TCP_NODELAY set
* Connected to 172.17.0.4 (172.17.0.4) port 8080 (#0)
> GET / HTTP/1.1
> Host: 172.17.0.4:8080
> User-Agent: curl/7.52.1
> Accept: */*
>
< HTTP/1.1 403 Forbidden
< content-length: 60
< content-type: text/plain
< date: Thu, 28 Nov 2019 06:27:39 GMT
< server: envoy
< x-envoy-upstream-service-time: 4
<
* Curl_http_done: called premature == 0
* Connection #0 to host 172.17.0.4 left intact
PERMISSION_DENIED:denyproductpagehandler.default:Not allowed
```

通过这个例子我们同时得知，istio 的黑白名单是基于 pod 的，禁止的是 pod 到 pod 的访问。

不过网络层依旧是可达的，网络层不在 istio 的管控范围内：

```sh
$ ping 172.17.0.4
PING 172.17.0.4 (172.17.0.4) 56(84) bytes of data.
64 bytes from 172.17.0.4: icmp_seq=1 ttl=64 time=0.367 ms
64 bytes from 172.17.0.4: icmp_seq=2 ttl=64 time=0.046 ms
```

### 按 IP 设置黑白名单

这是一个 ip 白名单的例子，只允许特定网段的 client 通过 ingressgateway 访问:

```sh
$ kubectl apply -f samples/bookinfo/policy/mixer-rule-deny-ip.yaml
```

通过下面三项配置，只允许 10.57.0.0/16 网段内的 ingressgateway 的访问：

```sh
apiVersion: config.istio.io/v1alpha2
kind: handler
metadata:
  name: whitelistip
spec:
  compiledAdapter: listchecker
  params:
    # providerUrl: ordinarily black and white lists are maintained
    # externally and fetched asynchronously using the providerUrl.
    overrides: ["10.57.0.0/16"]  # overrides provide a static list
    blacklist: false
    entryType: IP_ADDRESSES
---
apiVersion: config.istio.io/v1alpha2
kind: instance
metadata:
  name: sourceip
spec:
  compiledTemplate: listentry
  params:
    value: source.ip | ip("0.0.0.0")
---
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: checkip
spec:
  match: source.labels["istio"] == "ingressgateway"
  actions:
  - handler: whitelistip
    instances: [ sourceip ]
```

这个规则需要说明一下，rule 限定只作用于 ingressgateway，会检查 ingressgateway 的源 IP 是否在白名单内，如果不在就拒绝访问。

我这里的 ingressgateway 的 IP 是 172.17.0.19，不在白名单内：

```sh
$ kubectl -n istio-system get pod -o wide  |grep ingressgateway
istio-ingressgateway-75d6d5fd99-m2jnf     1/1     Running     1          6d2h   172.17.0.19 
```

通过该 ingressgateway 访问服务会被禁止：

![访问被禁止](../img/istio/ipdeny.png)

编辑 handler whitelistip，将 172.17 网段加入白名单后，就可以访问了：

```sh
$ kubectl edit handler whitelistip
...省略...
spec:
  compiledAdapter: listchecker
  params:
    blacklist: false
    entryType: IP_ADDRESSES
    overrides:
    - 10.57.0.0/16
    - 172.17.0.0/16
...
```

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
[3]: https://istio.io/docs/tasks/policy-enforcement/denial-and-list/ "Denials and White/Black Listing"

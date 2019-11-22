<!-- toc -->
# istio 的流量切换功能（类似于灰度发布）

istio 好像没有金丝雀以及国内常说的灰度发布概念，它的 [Traffic Shifting][2] 可以实现同样的效果，实现思路是在 [VirtualService](./vsvc.md) 中配置多个 destination，通过设置每个 destination 的权重，影响流量的分配。

## HTTP 请求的切换

编辑名为 reviews 的 VirtualService，为默认的 route 设置两个 destination，权重都是 50%：

```yaml
$ kubectl edit vs reviews
....省略...
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v3
      weight: 50
```

然后重复刷新页面，会发现页面的 reviews 部分在两个版本之间切换。

## TCP 请求的切分

TCP 请求可以按照 destination 的权重分配，设置方法类似，见 [TCP Traffic Shifting][4]。

## 更多配置

* [RouteDestination][3]

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://istio.io/docs/tasks/traffic-management/traffic-shifting/ "traffic-shifting"
[3]: https://istio.io/docs/reference/config/networking/virtual-service/#RouteDestination "RouteDestination"
[4]: https://istio.io/docs/tasks/traffic-management/tcp-traffic-shifting/ "TCP Traffic Shifting"

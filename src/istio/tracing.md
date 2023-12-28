<!-- toc -->
# istio 的链路跟踪（调用链）

istio 自动为 pod 发出的 http 请求添加追踪 headers，要实现完整的 [链路跟踪][2]，收到请求的应用程序在访问依赖的服务时，需要把请求头中的 headers 带到下一个请求中。

istio 注入的 headers：

```sh
x-request-id
x-b3-traceid
x-b3-spanid
x-b3-parentspanid
x-b3-sampled
x-b3-flags
x-ot-span-context
```

## 追踪 headers 

istio 自动为 pod 发出的 http 请求添加追踪 headers：

```sh
$ curl 10.101.26.165:8000
{
    "RemoteAddr": "127.0.0.1:54854",
    "Method": "GET",
    "Host": "10.101.26.165:8000",
    "RequestURI": "/",
    "Header": {
        "Accept": [
            "*/*"
        ],
        "Content-Length": [
            "0"
        ],
        "User-Agent": [
            "curl/7.61.1"
        ],
        "X-B3-Sampled": [
            "1"
        ],
        "X-B3-Spanid": [
            "cf8b6544e0a1b64c"
        ],
        "X-B3-Traceid": [
            "3775c9f1d6b7442acf8b6544e0a1b64c"
        ],
        "X-Forwarded-Proto": [
            "http"
        ],
        "X-Request-Id": [
            "81360a15-3ac5-9b9c-89ad-4baada513160"
        ]
    },
    "Body": ""
```

怎样把这些 headers 带到下一个请求，取决于应用程序的开发语言，例如 java 可以这样做：

```java
@GET
@Path("/reviews/{productId}")
public Response bookReviewsById(@PathParam("productId") int productId,
                            @HeaderParam("end-user") String user,
                            @HeaderParam("x-request-id") String xreq,
                            @HeaderParam("x-b3-traceid") String xtraceid,
                            @HeaderParam("x-b3-spanid") String xspanid,
                            @HeaderParam("x-b3-parentspanid") String xparentspanid,
                            @HeaderParam("x-b3-sampled") String xsampled,
                            @HeaderParam("x-b3-flags") String xflags,
                            @HeaderParam("x-ot-span-context") String xotspan) {

  if (ratings_enabled) {
    JsonObject ratingsResponse = getRatings(Integer.toString(productId), user, xreq, xtraceid, xspanid, xparentspanid, xsampled, xflags, xotspan);
```

## 设置采样频率

istio 支持调用链的抽检，调用链采样频率在 istio-pilot 中设置，默认是 100%：

```sh
$ kubectl -n istio-system edit deploy istio-pilot
...
        - name: PILOT_TRACE_SAMPLING
          value: "100"
...
```

## 查看调用链

用 [istio 操作命令](./command.md)  中的方法查看：

```sh
$ ./istioctl d jaeger
http://localhost:50887
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://istio.io/docs/tasks/observability/distributed-tracing/overview/ "Distributed Tracing"

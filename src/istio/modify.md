<!-- toc -->
# istio 的请求改写功能

istio 可以按照用户的定义改写请求，譬如增加请求头、改写 uri、请求重定向等。

[Control Headers and Routing][2] 演示了改写请求的方法同时也演示了 istio 的 adpater 的用法。

## 启动 adpater

keyval adpater 是一个 key:value 服务，输入 key 返回 value，如果 key 不存在返回 NOT_FOUND。

启动 keyval 容器：

```sh
$ kubectl run keyval --image=gcr.io/istio-testing/keyval:release-1.1 --namespace istio-system --port 9070 --expose
```

确定 keyval 创建成功：

```sh
$ kubectl -n istio-system get pod |grep keyval 
keyval-57f7c4b4cc-t5xw7                   1/1     Running     0          2m51s
```

把 keyval 封装成 istio 的 adapter：

```sh
$ kubectl apply -f samples/httpbin/policy/keyval-template.yaml
$ kubectl apply -f samples/httpbin/policy/keyval.yaml
```

keyval-template.yaml 的类型是 template，keyval.yaml 的类型是 adapter， 现在只能判断这就是在 handler 中使用的 adapter。

adapter 的运行、衔接机制现在还不清楚（2019-11-27 19:25:30）。adapter 显然是 istio 的重点特性，istio 的灵活性很大一部分来自于 apdater。

## 创建 handler

handler 引用了前面的创建的 adpater，并设置了 params（一个 K-V 查找表）

```yaml
$ kubectl apply -f - <<EOF
apiVersion: config.istio.io/v1alpha2
kind: handler
metadata:
  name: keyval
  namespace: istio-system
spec:
  adapter: keyval
  connection:
    address: keyval:9070
  params:
    table:
      jason: admin
EOF
```

## 创建 instance

instance 引用了前面创建的 template：

```yaml
$ kubectl apply -f - <<EOF
apiVersion: config.istio.io/v1alpha2
kind: instance
metadata:
  name: keyval
  namespace: istio-system
spec:
  template: keyval
  params:
    key: request.headers["user"] | ""
EOF
```

这个 instance 的含义使用请求头中的 user 头的值作为 key，从 keyval 获取价值。

前面的 adpater 中设置的 jason 对应的只为 admin，后面会用到这个 key。

## 创建 rule

rule 将前面创建的 handler 和 instance 绑定，设置了处理动作（注入 user-group header）：

```yaml
$ kubectl apply -f - <<EOF
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: keyval
  namespace: istio-system
spec:
  actions:
  - handler: keyval.istio-system
    instances: [ keyval ]
    name: x
  requestHeaderOperations:
  - name: user-group
    values: [ x.output.value ]
EOF
```

这个 handler 的含义是读取前面创建的 instance 的值（x），然后在请求头中添加一个 user-group header，字段值为 instaces 的值。

## http 头改写效果

带上 user:jason 请求头，访问 http-record 服务（一个回显请求的 http 服务）：

```yaml
curl  -H "user:jason"   10.101.26.165:8000
{
    "RemoteAddr": "127.0.0.1:41098",
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
        "User": [
            "jason"
        ],
        "User-Agent": [
            "curl/7.61.1"
        ],
        "User-Group": [
            "admin"
        ],
        "X-B3-Sampled": [
            "1"
        ],
        "X-B3-Spanid": [
            "51eb9b405f2c1207"
        ],
        "X-B3-Traceid": [
            "a63afc71120d234051eb9b405f2c1207"
        ],
        "X-Forwarded-Proto": [
            "http"
        ],
        "X-Request-Id": [
            "877e616c-7abc-93f4-913d-7dea33b3144e"
        ]
    },
    "Body": ""
}
```

可以看到服务端接收的请求头中多了一个 "User-Group”。

## 修改 uri 

修改 uri 的操作和注入请求头类似：

```yaml
$ kubectl apply -f - <<EOF
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: keyval
  namespace: istio-system
spec:
  match: source.labels["istio"] == "ingressgateway"
  actions:
  - handler: keyval.istio-system
    instances: [ keyval ]
  requestHeaderOperations:
  - name: :path
    values: [ '"/status/418"' ]
EOF
```

上面的定义中使用了 match，只影响到 ingressgateway 的请求，通过 ingressgateway 访问，uri 被改写：

```sh
$ curl -H "user: jason"  -H "Host: http-record.example" 192.168.99.100:31380
{
    "RemoteAddr": "127.0.0.1:50684",
    "Method": "GET",
    "Host": "http-record.example",
    "RequestURI": "/status/418",
...省略...
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://istio.io/docs/tasks/policy-enforcement/control-headers/ "Control Headers and Routing"

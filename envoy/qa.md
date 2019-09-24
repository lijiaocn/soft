<!-- toc -->

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

# Envoy 使用过程中的常见问题

这里是 Envoy 使用过程中常遇到的问题。

## Listener 的端口不可以重复

如果为两个 listener 配置了相同的监听端口，后一个 listener 会失败：

```sh
[2019-09-19 08:38:17.655][1][info][main] [source/server/server.cc:516] starting main dispatch loop
[2019-09-19 08:38:17.664][1][critical][config] [source/server/listener_manager_impl.cc:684] listener 'listener_1' failed to listen on address '0.0.0.0:81' on worker
```

listener 的监听端口不能重复。

## Route 的域名不能重复

Http 路由中的域名不能重复，如果两个 Route 中配置了相同的域名，或者一个 Route 中配置了两个相同的域名，那么配置会下发失败，envoy 打印下面的日志：

```sh
[2019-09-10 08:41:35.917][1][warning][config] [source/common/config/grpc_mux_subscription_impl.cc:72] gRPC config for type.googleapis.com/envoy.api.v2.Listener rejected: Error adding/updating listener(s) TCP-80: Only unique values for domains are permitted. Duplicate entry of domain echo.example
[2019-09-10 08:41:35.919][1][warning][config] [source/common/config/grpc_mux_subscription_impl.cc:72] gRPC config for type.googleapis.com/envoy.api.v2.Listener rejected: Error adding/updating listener(s) TCP-80: Only unique values for domains are permitted. Duplicate entry of domain echo.example
[2019-09-10 08:41:35.921][1][warning][config] [source/common/config/grpc_mux_subscription_impl.cc:72] gRPC config for type.googleapis.com/envoy.api.v2.Listener rejected: Error adding/updating listener(s) TCP-80: Only unique values for domains are permitted. Duplicate entry of domain echo.example
[2019-09-10 08:41:35.922][1][warning][config] [source/common/config/grpc_mux_subscription_impl.cc:72] gRPC config for type.googleapis.com/envoy.api.v2.Listener rejected: Error adding/updating listener(s) TCP-80: Only unique values for domains are permitted. Duplicate entry of domain echo.example
```

## Route 的 Prefix 可以重复 

Route 的 Prefix 可以重复，试验结果表明排在前面的 Prefix 被采用，下面的例子前缀为 /abc 的请求被转发到 STD@kube-cluster-1@demo-webshell@webshell@80 ：

```json
"routes": [
  {
    "route": {
      "cluster": "STD@kube-cluster-1@demo-webshell@webshell@80",
      "host_rewrite": "envoy.echo.example"
    },
    "match": {
      "case_sensitive": false,
      "prefix": "/abc"
    }
  },
  {
    "route": {
      "cluster": "STD@kube-cluster-1@demo-echo@echo@8080",
      "host_rewrite": "envoy.echo.example"
    },
    "match": {
      "prefix": "/abc",
      "case_sensitive": false
    }
  }
],
```

## 参考

<!-- toc -->
# go-controller-plane 的基本用法 

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

[go-control-plane][1] 是一个框架，提供了配置下发接口，在它的基础上根据自己的需要开发其它功能。

这里简单说明下用法，下一节有详细的例子，注意 **动态配置与 envoy 的配对** 的方法。

## 准备开发环境

用 [go mod][2] 引用 go-control-plane：

```sh
mkdir envoydev 
cd envoydev
go mod init github.com/introclass/go-code-example/envoydev
go get github.com/envoyproxy/go-control-plane@v0.8.3
```

## 使用示例

go-control-plane 中给出了一个简单的例子，snapshotCache 中存放要下发配置，将通过它创建的 server 与 grpcServer 绑定。envoy 通过 grpc 服务获取配置： 

```go
import (
    "net"

    api "github.com/envoyproxy/go-control-plane/envoy/api/v2"
    discovery "github.com/envoyproxy/go-control-plane/envoy/service/discovery/v2"
    "github.com/envoyproxy/go-control-plane/pkg/cache"
    xds "github.com/envoyproxy/go-control-plane/pkg/server"
)

func main() {
    snapshotCache := cache.NewSnapshotCache(false, hash{}, nil)
    server := xds.NewServer(snapshotCache, nil)
    grpcServer := grpc.NewServer()
    lis, _ := net.Listen("tcp", ":8080")

    discovery.RegisterAggregatedDiscoveryServiceServer(grpcServer, server)
    api.RegisterEndpointDiscoveryServiceServer(grpcServer, server)
    api.RegisterClusterDiscoveryServiceServer(grpcServer, server)
    api.RegisterRouteDiscoveryServiceServer(grpcServer, server)
    api.RegisterListenerDiscoveryServiceServer(grpcServer, server)
    go func() {
        if err := grpcServer.Serve(lis); err != nil {
            // error handling
        }
    }()
}
```

## 动态配置与 envoy 的配对

动态配置是关联到 envoy 的 ，每个 envoy 的动态配置都要单独维护。

在 envoy 的配置文件中有一段 node 配置，标注了当前 envoy 所属的 cluster 和 id，envoy 只接受使用同样的 cluster 和 id 的动态配置：

```yaml
node:
  id: "envoy-64.58"
  cluster: "test"
...
```

使用 go-control-plane 实现的控制平面要为每个 envoy 实例维护一份配置：

```go
node := &core.Node{
    Id:      "envoy-64.58",
    Cluster: "test",
}

node_config := &NodeConfig{
    node:      node,
    endpoints: []cache.Resource{}, //[]*api_v2.ClusterLoadAssignment
    clusters:  []cache.Resource{}, //[]*api_v2.Cluster
    routes:    []cache.Resource{}, //[]*api_v2.RouteConfiguration
    listeners: []cache.Resource{}, //[]*api_v2.Listener
}
```

## 参考

[1]: https://github.com/envoyproxy/go-control-plane "go-control-plane"
[2]: https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2019/05/05/go-modules.html "Go Modules：Go 1.11和1.12引入的依赖包管理方法"

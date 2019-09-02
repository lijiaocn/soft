<!-- toc -->
# go-control-plane 下发配置示例——代码说明

示例代码是 [xds.go][3]，这是一段超级简单、简陋的代码，只用来演示如何下发配置，下面是 xds.go 代码的说明，完整代码见 [xds.go][3]。

## NodeConfig

NodeConfig 中存放一个 envoy 的所有配置：

```go
+NodeConfig : struct
    [fields]
   -clusters : []cache.Resource
   -endpoints : []cache.Resource
   -listeners : []cache.Resource
   -node : *core.Node
   -routes : []cache.Resource
    [methods]
   +ID(node *core.Node) : string
```

代码中创建了下面的 node_config，它的 Id 和 Cluster 与 [envoy-0-example.yaml][2] 中的配置是对应的：

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

## 生成配置

下面这些函数用来生成要下发的配置，Cluster_ADS() 生成使用 ADS 发现 Endpoint 的 Cluster，对应的 endpoint 配置用 EDS() 生成，Listener 中动态发现的 Route，用 Route() 生成。

```go
+Cluster_ADS(name string) : *api_v2.Cluster
+Cluster_EDS(name string, edsCluster []string, edsName string) : *api_v2.Cluster
+Cluster_STATIC(name string, addrs []ADDR) : *api_v2.Cluster
+EDS(cluster string, addrs []ADDR) : *api_v2.ClusterLoadAssignment

+Listener_ADS(name string, port uint32, routeName string) : *api_v2.Listener
+Listener_RDS(name string, port uint32, routeName string, rdsCluster []string) : *api_v2.Listener
+Listener_STATIC(name string, port uint32, host, prefix, toCluster string) : *api_v2.Listener
+Route(name, host, prefix, toCluster string) : *api_v2.RouteConfiguration
```

## 配置下发

用下面的方式启动 grpc 服务：

```go
snapshotCache := cache.NewSnapshotCache(false, NodeConfig{}, nil)
server := xds.NewServer(snapshotCache, nil)
grpcServer := grpc.NewServer()
lis, _ := net.Listen("tcp", ":5678")

discovery.RegisterAggregatedDiscoveryServiceServer(grpcServer, server)
api_v2.RegisterEndpointDiscoveryServiceServer(grpcServer, server)
api_v2.RegisterClusterDiscoveryServiceServer(grpcServer, server)
api_v2.RegisterRouteDiscoveryServiceServer(grpcServer, server)
api_v2.RegisterListenerDiscoveryServiceServer(grpcServer, server)

go func() {
	if err := grpcServer.Serve(lis); err != nil {
		// error handling
	}
}()
```

Update_SnapshotCache() 将 node_config 填充到  snapshotCache 中，填充时指定配置的版本号，只要配置号发生了变化就被认为是需要更新的配置，没有递增、回退的概念。

```go
func Update_SnapshotCache(s cache.SnapshotCache, n *NodeConfig, version string) {
	err := s.SetSnapshot(n.ID(n.node), cache.NewSnapshot(version, n.endpoints, n.clusters, n.routes, n.listeners))
	if err != nil {
		glog.Error(err)
	}
}
```

## 配置下发举例

下发一个通过 xds_cluster 发现 endpoint 的 cluster，cluster 名为 Cluster_With_Dynamic_Endpoint。

```go
{
	clusterName := "Cluster_With_Dynamic_Endpoint"

	fmt.Printf("\nEnter to update version 2: %s", clusterName)
	_, _ = fmt.Scanf("\n", &input)

	var addrs []ADDR
	addrs = append(addrs, ADDR{
		Address: "127.0.0.1",
		Port:    8082,
	})

	point := EDS(clusterName, addrs)
	node_config.endpoints = append(node_config.endpoints, point)

	var edsCluster []string
	edsCluster = append(edsCluster, "xds_cluster") //静态的配置的 cluster

	edsName := clusterName
	cluster := Cluster_EDS(clusterName, edsCluster, edsName)
	node_config.clusters = append(node_config.clusters, cluster)

	Update_SnapshotCache(snapshotCache, node_config, "2")
	fmt.Printf("ok")
}
```

## 参考

[1]: https://github.com/introclass/go-code-example/tree/master/envoydev/xds  "github.com/introclass/go-code-example/envoydev/xds"
[2]: https://github.com/introclass/go-code-example/blob/master/envoydev/xds/envoy-docker-run/envoy-0-example.yaml "envoy-0-example.yaml"
[3]: https://github.com/introclass/go-code-example/blob/master/envoydev/xds/xds.go "xds.go"

<!-- toc -->
# go-control-plane 下发配置示例——运行和效果


## envoy 初始状态

envoy 启动之后，通过 admin 地址（浏览器打开 IP:9901 ） 查看 envoy 的当前配置以及内部情况：

![envoy 的 admin 界面](/img/envoy/envoy-admin.png)

envoy 的当前配置通过 IP:9901/config_dump 查看，通过该地址查询的配置文件是分为几段的：

* 第一段为 "@type": "type.googleapis.com/envoy.admin.v2alpha.BootstrapConfigDump"，是 envoy 启动时的配置
* 第二段为 "@type": "type.googleapis.com/envoy.admin.v2alpha.ClustersConfigDump"，是 cluster 配置，这里现在只有静态配置的 ads_cluster
* 第三段为 "@type": "type.googleapis.com/envoy.admin.v2alpha.ListenersConfigDump"，第三段现在是空的

![envoy 的初始化配置界面](/img/envoy/envoy-config-init.png)

下发 listener 和 route 后会多出两段配置，里面是动态下发的 route：

* "@type": "type.googleapis.com/envoy.admin.v2alpha.ScopedRoutesConfigDump" 
* "@type": "type.googleapis.com/envoy.admin.v2alpha.RoutesConfigDump" 

![envoy 的配置变化](/img/envoy/envoy-config-after.png)

endpoint 在配置页面中看不到，要到 IP:9901/clusters 中查看：

![envoy 的初始化endpoint界面](/img/envoy/envoy-ed-init.png)

## 启动 xds 

演示实现的控制平面的功能如下，每按一次回车，下发一组配置：

```sh
$ ./xds
Enter to update version 1: Cluster_With_Static_Endpoint
ok
Enter to update version 2: Cluster_With_Dynamic_Endpoint
ok
Enter to update version 3: Cluster_With_ADS_Endpoint
ok
Enter to update version 4: Listener_With_Static_Route
ok
Enter to update version 5: Listener_With_Dynamic_Route
ok
Enter to update version 6: Listener_With_ADS_Route
ok
Enter to exit: ^C
```

## 使用静态 endpoint 的 cluster

增加一个地址为 127.0.0.1:8081 的 cluster，下发后多出一个名为  Cluster_With_Static_Endpoint 的 cluster：

![envoy中下发的静态cluster](/img/envoy/envoy-static-cluster.png)

对应代码：

```go
{
    clusterName := "Cluster_With_Static_Endpoint"
    fmt.Printf("Enter to update version 1: %s", clusterName)
    _, _ = fmt.Scanf("\n", &input)

    var addrs []ADDR
    addrs = append(addrs, ADDR{
        Address: "127.0.0.1",
        Port:    8081,
    })
    cluster := Cluster_STATIC(clusterName, addrs)
    node_config.clusters = append(node_config.clusters, cluster)
    Update_SnapshotCache(snapshotCache, node_config, "1")
    fmt.Printf("ok")
}
```

## 使用 eds 发现 endpoint 的 cluster

增加一个地址为 127.0.0.1:8082 的 cluster，下发后多出一个名为 Cluster_With_Dynamic_Endpoint 的 cluster：

![envoy中下发的静态cluster](/img/envoy/envoy-cluster-eds.png)

cluster 没有直接配置 endpoint，而是指定从 xds_cluster 中获取，在 IP:9901/clusters 中可以看到 endpoint：

![envoy中动态获取的endpoints](/img/envoy/envoy-dynamic-cls-ep.png)

对应代码： 

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

## 使用 ads 发现 endpoint 的 cluster

增加一个地址为 127.0.0.1:8083 的 cluster，下发后多出一个名为 Cluster_With_ADS_Endpoint 的 cluster，ads 的配置和 xds 不同，不需要指定 cluster，只需要声明 ADS 即可：

![使用 ads 发现 endpoint 的 cluster](/img/envoy/envoy-cls-ads.png)


对应代码：

```go
{
    clusterName := "Cluster_With_ADS_Endpoint"
    fmt.Printf("\nEnter to update version 3: %s", clusterName)
    _, _ = fmt.Scanf("\n", &input)

    var addrs []ADDR
    addrs = append(addrs, ADDR{
        Address: "127.0.0.1",
        Port:    8083,
    })

    edsName := clusterName
    point := EDS(edsName, addrs)
    node_config.endpoints = append(node_config.endpoints, point)

    cluster := Cluster_ADS("Cluster_With_ADS_Endpoint")
    node_config.clusters = append(node_config.clusters, cluster)

    Update_SnapshotCache(snapshotCache, node_config, "3")
    fmt.Printf("ok")
}
```

## 使用静态路由的 listener

前面只下发了 cluster，没有下发 listener，无法访问 cluster。要访问 cluster 必须配置一个指向它的 listener，下发一个监听 84 端口，转发到 127.0.0.1:8084 的 listener：

![使用 ads 发现 endpoint 的 cluster](/img/envoy/envoy-listener.png)

对应代码，转发规则为 Host 是 webshell.com，prefix 是 /abc：

```go
{
    listenerName := "Listener_With_Static_Route"
    fmt.Printf("\nEnter to update version 4: %s", listenerName)
    _, _ = fmt.Scanf("\n", &input)

    clusterName := "Listener_With_Static_Route_Target_Cluster"
    var addrs []ADDR
    addrs = append(addrs, ADDR{
        Address: "127.0.0.1",
        Port:    8084,
    })
    cluster := Cluster_STATIC(clusterName, addrs)
    node_config.clusters = append(node_config.clusters, cluster)

    lis := Listener_STATIC(listenerName, 84, "webshell.com", "/abc", clusterName)
    node_config.listeners = append(node_config.listeners, lis)

    Update_SnapshotCache(snapshotCache, node_config, "4")
    fmt.Printf("ok")
}
```

下发后可以访问 cluster：

```sh
$ curl -v  -H "Host: webshell.com" 127.0.0.1:84/abc
* About to connect() to 127.0.0.1 port 84 (#0)
*   Trying 127.0.0.1...
* Connected to 127.0.0.1 (127.0.0.1) port 84 (#0)
> GET /abc HTTP/1.1
> User-Agent: curl/7.29.0
> Accept: */*
> Host: webshell.com
>
< HTTP/1.1 200 OK
< server: envoy
< date: Thu, 15 Aug 2019 07:44:51 GMT
< content-type: application/octet-stream
< content-length: 3
< last-modified: Thu, 15 Aug 2019 02:40:31 GMT
< x-envoy-upstream-service-time: 1
<
aa
* Connection #0 to host 127.0.0.1 left intact
```

## 参考

[1]: https://github.com/introclass/go-code-example/tree/master/envoydev/xds  "github.com/introclass/go-code-example/envoydev/xds"
[2]: https://github.com/introclass/go-code-example/blob/master/envoydev/xds/envoy-docker-run/envoy-0-example.yaml "envoy-0-example.yaml"
[3]: https://github.com/introclass/go-code-example/blob/master/envoydev/xds/xds.go "xds.go"

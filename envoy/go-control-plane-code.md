<!-- toc -->
# go-control-plane 下发配置实例

这里演示一个用 go-control-plane 下发配置的例子。

## 准备 envoy

用下面的配置文件启动一个 envoy ，id 为 envoy-64.58，cluster 为 test，admin 端口为 9901，使用 ADS，ADS 服务地址为 127.0.0.1: 5678（这里例子中 ADS 服务和和 envoy 在同一台机器上，如果在不同的机器上，换成 ADS 的实际地址）：

```yaml
node:
  id: "envoy-64.58"
  cluster: "test"
#runtime:
#  symlink_root: /srv/runtime/current
#  subdirectory: envoy
#  override_subdirectory: envoy_override
watchdog:
  miss_timeout: 0.2s
  megamiss_timeout: 1s
  kill_timeout: 0s
  multikill_timeout: 0s
flags_path: /etc/envoy/flags/
stats_flush_interval: 5s
stats_config:
  use_all_default_tags: true
stats_sinks:
  name: envoy.stat_sinks.hystrix
  config:
    num_buckets: 10
admin:
  access_log_path: /var/log/envoy/admin_access.log
  profile_path: /var/log/envoy/envoy.prof
  address:
    socket_address:
      protocol: TCP
      address: 0.0.0.0
      port_value: 9901
dynamic_resources:
  ads_config:
    api_type: GRPC
    grpc_services:
      envoy_grpc:
        cluster_name: ads_cluster
  cds_config: {ads: {}}
  lds_config: {ads: {}}
static_resources:
  clusters:
  - name: ads_cluster
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    http2_protocol_options: {}
    load_assignment:
      cluster_name: ads_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 5678
```

启动，envoy 版本为 v1.11.0：

```sh
IMAGE=envoyproxy/envoy:v1.11.0
docker run -idt --network=host -v `pwd`/envoy-static.yaml:/etc/envoy/envoy.yaml -v `pwd`/log:/var/log/envoy $IMAGE
```

envoy 启动之后，可以通过 admin 地址（浏览器打开 IP地址:9901 ） 查看 envoy 的当前配置以及内部情况：

![envoy 的 admin 界面](/img/envoy/envoy-admin.png)

envoy 的当前配置通过 IP地址:9901/config_dump 查看，通过改地址查询的配置文件是分为几段的，

* 第一段类型为 "@type": "type.googleapis.com/envoy.admin.v2alpha.BootstrapConfigDump"，是 envoy 启动时使用的配置
* 第二段为 "@type": "type.googleapis.com/envoy.admin.v2alpha.ClustersConfigDump"，是 cluster 配置
这里现在只有静态配置的 ads_cluster
* 第三段是 "@type": "type.googleapis.com/envoy.admin.v2alpha.ListenersConfigDump"，第三段现在是空的

![envoy 的初始化配置界面](/img/envoy/envoy-config-init.png)

## ads 实现代码

这里实现一个最简单的 ads，这个 ads 只向一个 envoy 下发程序中写死的配置：

* 启动后键入回车，下发一个名为 listener_with_dynamic_route_port_9002 的 listener
* listener 中使用了名为 envoy.http_connection_manager 的 network filter
* 上述 network filter 通过 ads 获取路由配置，路由配置的名称为 ads_route
* 动态下发一条 route，route 内容为：将匹配 "/" 的 http 请求转发给名为 cluster_with_ads_endpoint  的 cluster，并将 Host 改写为 webshell.com

下面代码是试验代码的删减版，cluster_with_ads_endpoint 是原始版本动态下发的，原始版本代码位于 [github.com/introclass/go-code-example/envoydev/xds][1]。

```go
// Create: 2018/12/29 18:32:00 Change: 2019/08/12 17:22:24
// FileName: main.go
// Copyright (C) 2018 lijiaocn <lijiaocn@foxmail.com>
//
// Distributed under terms of the GPL license.

package main

import (
	"fmt"
	"net"

	api_v2 "github.com/envoyproxy/go-control-plane/envoy/api/v2"
	"github.com/envoyproxy/go-control-plane/envoy/api/v2/core"
	"github.com/envoyproxy/go-control-plane/envoy/api/v2/listener"
	"github.com/envoyproxy/go-control-plane/envoy/api/v2/route"
	http_router "github.com/envoyproxy/go-control-plane/envoy/config/filter/http/router/v2"
	http_conn_manager "github.com/envoyproxy/go-control-plane/envoy/config/filter/network/http_connection_manager/v2"
	discovery "github.com/envoyproxy/go-control-plane/envoy/service/discovery/v2"
	"github.com/envoyproxy/go-control-plane/pkg/cache"
	xds "github.com/envoyproxy/go-control-plane/pkg/server"
	"github.com/envoyproxy/go-control-plane/pkg/util"
	proto_type "github.com/gogo/protobuf/types"
	"github.com/golang/glog"
	"google.golang.org/grpc"
)

type NodeConfig struct {
	node      *core.Node
	endpoints []cache.Resource //[]*api_v2.ClusterLoadAssignment
	clusters  []cache.Resource //[]*api_v2.Cluster
	routes    []cache.Resource //[]*api_v2.RouteConfiguration
	listeners []cache.Resource //[]*api_v2.Listener
}

//implement cache.NodeHash
func (n NodeConfig) ID(node *core.Node) string {
	return node.GetId()
}


func main() {
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

	input := ""

	fmt.Printf("\nEnter to update version 6: ADD_Listener_With_ADS_Route")
	_, _ = fmt.Scanf("\n", &input)
	ADD_Listener_With_ADS_Route(node_config)
	Update_SnapshotCache(snapshotCache, node_config, "6")
	fmt.Printf("ok")

	fmt.Printf("\nEnter to exit: ")
	_, _ = fmt.Scanf("\n", &input)
}

func ADD_Listener_With_ADS_Route(n *NodeConfig) {

	r := &route.Route{
		Match: &route.RouteMatch{
			PathSpecifier: &route.RouteMatch_Prefix{
				Prefix: "/",
			},
			CaseSensitive: &proto_type.BoolValue{
				Value: false,
			},
		},
		Action: &route.Route_Route{
			Route: &route.RouteAction{
				ClusterSpecifier: &route.RouteAction_Cluster{
					Cluster: "cluster_with_ads_endpoint",
				},
				HostRewriteSpecifier: &route.RouteAction_HostRewrite{
					HostRewrite: "webshell.com",
				},
			},
		},
	}

	routes := make([]*route.Route, 0)
	routes = append(routes, r)

	virtualHost := &route.VirtualHost{
		Name: "local",
		Domains: []string{
			"ads.webshell.com",
		},
		Routes: routes,
	}

	virtualHosts := make([]*route.VirtualHost, 0)
	virtualHosts = append(virtualHosts, virtualHost)

	routeConfig := &api_v2.RouteConfiguration{
		Name:         "ads_route",
		VirtualHosts: virtualHosts,
	}

	n.routes = append(n.routes, routeConfig)

	http_filter_router_ := &http_router.Router{
		DynamicStats: &proto_type.BoolValue{
			Value: true,
		},
	}

	http_filter_router, err := util.MessageToStruct(http_filter_router_)
	if err != nil {
		glog.Error(err)
		return
	}

	httpFilter := &http_conn_manager.HttpFilter{
		Name: "envoy.router",
		ConfigType: &http_conn_manager.HttpFilter_Config{
			Config: http_filter_router,
		},
	}

	httpFilters := make([]*http_conn_manager.HttpFilter, 0)
	httpFilters = append(httpFilters, httpFilter)

	listen_filter_http_conn_ := &http_conn_manager.HttpConnectionManager{
		StatPrefix: "ingress_http",
		RouteSpecifier: &http_conn_manager.HttpConnectionManager_Rds{
			Rds: &http_conn_manager.Rds{
				RouteConfigName: "ads_route",
				ConfigSource: &core.ConfigSource{
					ConfigSourceSpecifier: &core.ConfigSource_Ads{
						Ads: &core.AggregatedConfigSource{}, //使用ADS
					},
				},
			},
		},
		HttpFilters: httpFilters,
	}

	listen_filter_http_conn, err := util.MessageToStruct(listen_filter_http_conn_)
	if err != nil {
		glog.Error(err)
		return
	}

	filter := &listener.Filter{
		Name: "envoy.http_connection_manager",
		ConfigType: &listener.Filter_Config{
			Config: listen_filter_http_conn,
		},
	}

	filters := make([]*listener.Filter, 0)
	filters = append(filters, filter)

	filterChain := &listener.FilterChain{
		Filters: filters,
	}

	filterChains := make([]*listener.FilterChain, 0)
	filterChains = append(filterChains, filterChain)

	socketAddr := &core.SocketAddress{
		Protocol: core.TCP,
		Address:  "0.0.0.0",
		PortSpecifier: &core.SocketAddress_PortValue{
			PortValue: 9002,
		},
	}

	addr := &core.Address{
		Address: &core.Address_SocketAddress{
			SocketAddress: socketAddr,
		},
	}

	lis := &api_v2.Listener{
		Name:         "listener_with_dynamic_route_port_9002",
		Address:      addr,
		FilterChains: filterChains,
	}

	n.listeners = append(n.listeners, lis)
}

func Update_SnapshotCache(s cache.SnapshotCache, n *NodeConfig, version string) {
	err := s.SetSnapshot(n.ID(n.node), cache.NewSnapshot(version, n.endpoints, n.clusters, n.routes, n.listeners))
	if err != nil {
		glog.Error(err)
	}
}

```

## 下发效果

用上面的 ads 下发配置后，刷新 IP:9901/config_dump 可以看到变化：

* 第三段 listener 不再是空的，其中有用 ads 下发的配置
* 多了两段配置： "@type": "type.googleapis.com/envoy.admin.v2alpha.ScopedRoutesConfigDump" 和 "@type": "type.googleapis.com/envoy.admin.v2alpha.RoutesConfigDump" ，里面是动态下发的 route

![envoy 的配置变化](/img/envoy/envoy-config-after.png)

## 参考

[1]: https://github.com/introclass/go-code-example/tree/master/envoydev/xds  "github.com/introclass/go-code-example/envoydev/xds"

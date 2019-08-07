# go-control-plane 中的 filter 定义

[go-control-plane][7] 是一个控制平面的开发框架，见之前的 [笔记][7]。

Listener 在 [go-control-plane/envoy/api/v2/lds.pb.go][8] 中定义，filter 的接口在 [envoy/api/v2/listener/listener.pb.go][9] 中定义，filter 的具体实现在 [go-control-plane/envoy/config/filter/][10]。

### listener_filters 的定义

先起作用的 listener_filters 的成员在 go-control-plane 中定义为 listener.ListenerFilter，如下：

```go
// go-control-plane/envoy/api/v2/lds.pb.go
ListenerFilters []*listener.ListenerFilter 

// go-control-plane/envoy/api/v2/listener/listener.pb.go
type ListenerFilter struct {
    Name string 
    ConfigType           isListenerFilter_ConfigType 
    XXX_NoUnkeyedLiteral struct{}                    
    XXX_unrecognized     []byte                      
    XXX_sizecache        int32                       
}
```

### filter_chains 的定义

后起作用的 filter_chains 的成员在 go-control-plane 中定义为 listener.FilterChain，成员 FilterChain 又包含一条 Filter 链：

```go
FilterChains []*listener.FilterChain 

type FilterChain struct {
    FilterChainMatch *FilterChainMatch 
    TlsContext *auth.DownstreamTlsContext 
    Filters []*Filter 
    UseProxyProto *types.BoolValue 
    Metadata *core.Metadata 
    TransportSocket      *core.TransportSocket 
    
    XXX_NoUnkeyedLiteral struct{}              
    XXX_unrecognized     []byte                
    XXX_sizecache        int32                 
}

type Filter struct {
    Name string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
    ConfigType           isFilter_ConfigType `protobuf_oneof:"config_type"`
    XXX_NoUnkeyedLiteral struct{}            `json:"-"`
    XXX_unrecognized     []byte              `json:"-"`
    XXX_sizecache        int32               `json:"-"`
}

```

## 在 go-control-plane 中填充 filter

以 listener_filters 的填充为例，ConfigType 就是填充的 filter，是一个接口类型的成员：

```go
// go-control-plane/envoy/api/v2/listener/listener.pb.go
type ListenerFilter struct {
    Name string 
    ConfigType           isListenerFilter_ConfigType 
    XXX_NoUnkeyedLiteral struct{}                    
    XXX_unrecognized     []byte                      
    XXX_sizecache        int32                       
}

// go-control-plane/envoy/api/v2/listener/listener.pb.go: 585
type isListenerFilter_ConfigType interface {
    isListenerFilter_ConfigType()
    Equal(interface{}) bool
    MarshalTo([]byte) (int, error)
    Size() int
}
```

ConfigType 的类型是一个很简单的接口，但需要 `特别注意` 满足该接口的不是 [go-control-plane/envoy/config/filter][10] 中每个 filter 的具体实现，而是两个超级简单的 struct： `ListenerFilter_Config` 和 `ListenerFilter_TypedConfig` 。

```go
// go-control-plane/envoy/api/v2/listener/listener.pb.go: 592
type ListenerFilter_Config struct {
    Config *types.Struct 
}
type ListenerFilter_TypedConfig struct {
    TypedConfig *types.Any
}

func (*ListenerFilter_Config) isListenerFilter_ConfigType()      {}
func (*ListenerFilter_TypedConfig) isListenerFilter_ConfigType() {}
```

从上面的代码可以看到这两个 struct 仅仅是定义了接口方法而已，函数体是空的。包含 filter 配置的成员 Config/TypedConfig 是 grpc 格式的数据 *types.Struct / *types.Any。

filter 的实现位于 [go-control-plane/envoy/config/filter][10] 目录 ，需要用 util.MessageToStruct() 方法转换成 *types.Struct，或者用 ptypes.MarshalAny() 转换成 *any.Any后，作为 Config 或 TypedConfig 的值。

filter_chains 的情况类似，对应的 struct 是 `Filter_Config` 和 `Filter_TypedConfig`。

以 [HTTP Connection Manager][2] 为例，填充操作如下：

```sh
// 创建一个 HttpConnectionManager
listen_filter_http_conn_ := &http_conn_manager.HttpConnectionManager{
    StatPrefix: "ingress_http",
    RouteSpecifier: &http_conn_manager.HttpConnectionManager_RouteConfig{
        RouteConfig: &api_v2.RouteConfiguration{
            Name:         "None",
            VirtualHosts: virtualHosts,
        },
    },
    HttpFilters: httpFilters,
}

// 转换成 *types.Struct
listen_filter_http_conn, err := util.MessageToStruct(listen_filter_http_conn_)
if err != nil {
    glog.Error(err)
    return
}

// 用转换得到 *types.Struct 构造 listener.Filter_Config，继而构造 Filter
filter := &listener.Filter{
    Name: "envoy.http_connection_manager",
    ConfigType: &listener.Filter_Config{
        Config: listen_filter_http_conn,
    },
}

//然后把 filter 放入 filterChains 的一个成员的 filters 链中。
filters = append(filters, filter)
filterChain := &listener.FilterChain{
    Filters: filters,
}

//最后将 filterChains 装入 listener
filterChains = append(filterChains, filterChain)
lis := &api_v2.Listener{
    Name:         "listener_with_static_route_port_9000",
    Address:      address,
    FilterChains: filterChains,
}
```

[1]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/listeners/listeners "Listeners"
[2]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/network/http_connection_manager/v2/http_connection_manager.proto#envoy-api-msg-config-filter-network-http-connection-manager-v2-httpconnectionmanager  "HTTP Connection Manager"
[3]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/address.proto#envoy-api-msg-core-address "core.Address"
[4]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-msg-listener-filterchain "listener.FilterChain"
[5]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/listener/listener.proto#envoy-api-msg-listener-listenerfilter  "listener.ListenerFilter"
[6]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/config#  "Extensions"
[7]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/29/envoy-07-features-2-dynamic-discovery.html#go-control-plane "go-control-plane"
[8]: https://github.com/envoyproxy/go-control-plane/blob/v0.8.4/envoy/api/v2/lds.pb.go "go-control-plane/envoy/api/v2/lds.pb.go"
[9]: https://github.com/envoyproxy/go-control-plane/blob/v0.8.4/envoy/api/v2/listener/listener.pb.go "envoy/api/v2/listener/listener.pb.go"
[10]: https://github.com/envoyproxy/go-control-plane/tree/v0.8.4/envoy/config/filter "go-control-plane/envoy/config/filter/"
[11]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/filter/filter "Filters"
[12]: https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/config#  "Extensions"
[13]: https://www.envoyproxy.io/docs/envoy/latest/configuration/configuration  "Configuration reference"

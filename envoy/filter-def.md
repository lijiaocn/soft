<!-- toc -->
# go-control-plane 中的 filter 定义与下发 

>视频讲解：[Envoy手把手入门视频讲解](https://study.163.com/course/courseMain.htm?share=2&shareId=400000000376006&courseId=1209487865&_trace_c_p_k2_=18c88dad391f427b9e40e0795d8d939d)

Filter 是填充在 listener 中作为 listener 配置下发的，listener 在 [go-control-plane/envoy/api/v2/lds.pb.go][8] 中定义，filter 的接口在 [envoy/api/v2/listener/listener.pb.go][9] 中定义，filter 的实现在 [go-control-plane/envoy/config/filter/][10] 中。

### listener_filters 的定义

Listener 中的 listener_filters 的在 go-control-plane 中定义如下：

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

Listener 中的 filter_chains 在 go-control-plane 中定义如下，其中FilterChain 继续包含 Filter 链：

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

## filter 的定义

filter 的定义在 [go-control-plane/envoy/config/filter][10] 中，不同类型的 filter 都有各自的定义。用 go-control-plane 下发时，需要用 util.MessageToStruct() 转换成 *types.Struct 类型，或者用 ptypes.MarshalAny() 转换成 * any.Any 类型后，填充到 Listener 中。

以 listener_filters 的为例，ListenerFilter 中的 ConfigType 是一个接口变量：

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

isListenerFilter_ConfigType 是一个很简单的接口，实现该接口的 struct 是 ListenerFilter_Config 和 ListenerFilter_TypedConfig 。

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

上面的 Config 和 TypedConfig 就是 filter 转化而成的。

filter_chains 的情况类似，对应的 struct 是 Filter_Config 和 Filter_TypedConfig。

## XX_Config 和 XX_TypedConfig 的区别

将 filter 填充到 listener 时，可以用  XX_Config 类型，也可以用 XX_TypedConfig 的类型，区别如下：

**XX_Config**：

```yaml
- name: envoy.http_connection_manager
  config:
    stat_prefix: ingress_http
    generate_request_id: true
    route_config:
      name: local_route
      virtual_hosts:
      - name: local_service
        domains: ["webshell.com"]
        routes:
        - match:
            prefix: "/"
          route:
            host_rewrite: webshell.com
            cluster: service_webshell
    http_filters:
    - name: envoy.router
      config:
        dynamic_stats: false
```

**XX_TypedConfig**：

```yaml
- name: envoy.http_connection_manager
  typed_config:
    "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
    stat_prefix: ingress_http
    route_config:
      name: local_route
      virtual_hosts:
      - name: local_service
        domains: ["*"]
        routes:
        - match:
            prefix: "/"
          route:
            host_rewrite: www.baidu.com
            cluster: service_baidu
    http_filters:
    - name: envoy.router
```

## HTTP Connection Manager 的填充过程

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

## 参考

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

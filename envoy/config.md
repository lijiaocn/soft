<!-- toc -->
# Envoy 的配置文件格式

Envoy 的 [API 文档](https://www.envoyproxy.io/docs/envoy/latest/api/api) 中，分别给出了每个配置项的格式，[《Envoy Proxy使用介绍教程（五）：envoy的配置文件完全展开介绍》](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/12/27/envoy-05-configfile.html) 将 envoy 1.8.0  的各个配置项格式组合了起来，呈现了 envoy 配置文件的完整轮廓：

```json
{
  "node": {
    "id": "...",
    "cluster": "...",
    "metadata": "{...}",
    "locality": "{...}",
    "build_version": "..."
  },
  "static_resources": {
    "listeners": [],
    "clusters": [],
    "secrets": []
  },
  "dynamic_resources": {
    "lds_config": "{...}",
    "cds_config": "{...}",
    "ads_config": "{...}"
  },
  "cluster_manager": {
    "local_cluster_name": "...",
    "outlier_detection": "{...}",
    "upstream_bind_config": "{...}",
    "load_stats_config": "{...}"
  },
  "hds_config": {
    "api_type": "...",
    "cluster_names": [],
    "grpc_services": [],
    "refresh_delay": "{...}",
    "request_timeout": "{...}",
    "rate_limit_settings": "{...}"
  },
  "flags_path": "...",
  "stats_sinks": [
    {
      "name": "...",
      "config": "{...}"
    }
  ],
  "stats_config": {
    "stats_tags": [],
    "use_all_default_tags": "{...}",
    "stats_matcher": "{...}"
  },
  "stats_flush_interval": "{...}",
  "watchdog": {
    "miss_timeout": "{...}",
    "megamiss_timeout": "{...}",
    "kill_timeout": "{...}",
    "multikill_timeout": "{...}"
  },
  "tracing": {
    "http": "{...}"
  },
  "rate_limit_service": {
    "grpc_service": "{...}"
  },
  "runtime": {
    "symlink_root": "...",
    "subdirectory": "...",
    "override_subdirectory": "..."
  },
  "admin": {
    "access_log_path": "...",
    "profile_path": "...",
    "address": "{...}"
  },
  "overload_manager": {
    "refresh_interval": "{...}",
    "resource_monitors": [],
    "actions": []
  }
}
```


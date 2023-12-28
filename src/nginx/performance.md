<!-- toc -->
# Nginx 性能优化时需要考虑的配置

做压测时了解到一些配置，在这里记录一下。调整的参数如下：

```conf
# 这不是一个完整的配置文件
worker_processes 2;
worker_cpu_affinity 0010 1000;
worker_rlimit_nofile  10340;
worker_priority -20;
events {
    worker_connections 10240;
    worker_aio_requests 32;
}
http{
...
    upstream webshell_upstream{
        keepalive 16;
        keepalive_timeout 60s;                # nginx 1.15.3
    }
    ...
    server {
        ...
        keepalive_requests  10000000;
        location / {
            ...
            proxy_socket_keepalive on;
            proxy_connect_timeout  60s;       # nginx 1.15.6
            proxy_send_timeout     60s;
            proxy_read_timeout     60s;
            proxy_http_version     1.1;
            proxy_set_header Connection "";
        }
    }
}
```

## Core functionality

[Core functionality](https://nginx.org/en/docs/ngx_core_module.html) 包含 nginx 的核心参数，与性能直接相关的有：

* [worker_processes](https://nginx.org/en/docs/ngx_core_module.html#worker_processes)：nginx 工作进程的数量，默认是`auto`
* [worker_cpu_affinity](https://nginx.org/en/docs/ngx_core_module.html#worker_cpu_affinity)：nginx工作进程绑定到指定的CPU核
* [worker_rlimit_nofile](https://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_nofile)：单个nginx worker可以打开的文件数，影响 nginx worker 建立的连接数量
* [worker_priority](https://nginx.org/en/docs/ngx_core_module.html#worker_priority)：nginx worker 进程在系统中的优先级，`-20~+20`，`-20`是最高优先级
* [worker_connections](https://nginx.org/en/docs/ngx_core_module.html#worker_connections)：单个 nginx worker 可以建立的连接数量，`不能高于` worker_rlimit_nofile。

## Module ngx_http_proxy_module 

[Module ngx_http_proxy_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html) 包含代理相关设置，与性能直接相关的有：

* [proxy_http_version](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_http_version)：nginx 到 upstream 中的 server 时使用的协议，默认是 http，不会建立长连接。设置成 http 1.1 ，用 [proxy_set_header](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header) 将 `Connection` 头去掉后，会启用长连接，性能显著提升。

## Module ngx_http_upstream_module   

[Module ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html) 包含 upstream 相关的参数，其中与性能直接相关的有：

* [keepalive](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) ：单个 worker 可以与 backend server 保持的空闲连接数。如果这个数值过小，nginx worker 会不停地与 backend server 新建连接、关闭连接。

* [keepalive_timeout](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive_timeout)：`1.15.3`中开始有的参数，设置 nginx worker 与 backend server 的空闲的连接保持时间。早期版本中没有这个参数，长连接是一直保持（可以用 keepalive_requests 的参数避免）。

* [keepalive_requests](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive_requests)：设置 nginx worker 与 backend server 建立的长连接中可以传输的请求数，超过后连接被 nginx 主动关闭。

关于长连接的设置可以参考：[Enable Keepalive connections in Nginx Upstream proxy configurations](https://ma.ttias.be/enable-keepalive-connections-in-nginx-upstream-proxy-configurations/)和[TCP keepalive overview](http://tldp.org/HOWTO/TCP-Keepalive-HOWTO/overview.html)。

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

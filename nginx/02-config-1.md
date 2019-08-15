# Nginx高性能配置

在做压测时，了解到一些配置，在这里记录一下。

调整的参数如下（下面只列出调整的参数，不是一个完成配置文件）：

```conf
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

[Core functionality](https://nginx.org/en/docs/ngx_core_module.html)中是nginx的核心参数，与性能直接相关的有：

[worker_processes](https://nginx.org/en/docs/ngx_core_module.html#worker_processes)是nginx工作进程的数量，默认是`auto`，nginx master进程自主决定启动几个nginx worker。

[worker_cpu_affinity](https://nginx.org/en/docs/ngx_core_module.html#worker_cpu_affinity)将nginx工作进程绑定到指定的CPU核。

[worker_rlimit_nofile](https://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_nofile)设置nginx worker可以打开的文件数，这个直接影响到nginx worker可以建立的连接数量。

[worker_priority](https://nginx.org/en/docs/ngx_core_module.html#worker_priority)设置nginx worker继承的优先级，范围是`-20~+20`， `-20`是最高优先级。

[worker_connections](https://nginx.org/en/docs/ngx_core_module.html#worker_connections)是每个nginx worker可以建立的连接数量，这个数值`不能高于`worker_rlimit_nofile。

## Module ngx_http_proxy_module 

[Module ngx_http_proxy_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)中是代理相关设置，与性能直接相关的有：

[proxy_http_version](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_http_version)设置nginx到upstream中的server时使用的协议，默认是http，不会建立长连接。修改为http 1.1使用长连接，性能会显著提高。使用http 1.1的时候，需要用[proxy_set_header](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header)将`Connection`头去掉，否则因为会Connection的值是`Close`,导致连接被断开。

## Module ngx_http_upstream_module   

[Module ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html)中upstream相关的参数，其中与性能直接相关的有：

[keepalive](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive)是每个worker可以与backend server保持的空闲连接数，如果这个数值过小，nginx worker会不停地与backend server新建连接、关闭连接。

[keepalive_timeout](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive_timeout)是`1.15.3`中开始有的参数，它的用途是设置nginx worker与backend server之间的空闲的连接保持时间。早期版本中没有这个参数，长连接是一直保持。（可以通过下面的keepalive_requests的参数避免）

[keepalive_requests](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive_requests)设置nginx worker与backend server建立的长连接中可以传输的请求数，超过设置，连接被nginx主动关闭。

关于长连接的设置可以参考：[Enable Keepalive connections in Nginx Upstream proxy configurations](https://ma.ttias.be/enable-keepalive-connections-in-nginx-upstream-proxy-configurations/)和[TCP keepalive overview](http://tldp.org/HOWTO/TCP-Keepalive-HOWTO/overview.html)。

## 参考

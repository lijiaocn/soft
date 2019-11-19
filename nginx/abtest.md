<!-- toc -->
# Nginx 的 A/B 测试功能

Nginx 的有一个 [ngx_http_split_clients_module][2] 模块，可以用来实现 A/B 测试。

## split_clients 使用方法

split_clients 指令的用途是根据设定条件设置变量的值，如下，定义了一个名为 selected_upstream 的变量：

```conf
split_clients "$date_gmt" $selected_upstream {
               50%          echo_upstream;
               50%          record_upstream;
}
```

第二个参数是 MurmurHash2 哈希算法的输入，可以使用 ngxin 变量，50% 是区间划分。上面的配置中，哈希输入是 date_gmt，变量 selected_upstream 的值 50% 的可能是 echo_upstream，50% 的可能是 record_upstream。

selected_upstream 的取值是按照指定配置变换的，可以用来实现一些特殊效果，例如下面的配置，请求会按照时间情况在两个 upstream（echo_upstream 和 record_upstream）之间分配。

```conf
upstream echo_upstream {
    server  127.0.0.1:9090;
    keepalive 1;
}

upstream record_upstream {
    server  127.0.0.1:9091;
    keepalive 1;
}

split_clients "$date_gmt" $selected_upstream {
               50%          echo_upstream;
               50%          record_upstream;
}

server {
    listen       9000;
    server_name  echo.example;
    keepalive_requests  1000;
    keepalive_timeout 60s;

    location / {
        proxy_pass  http://$selected_upstream;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

## 参考

1. [李佶澳的博客][1]
2. [Module ngx_http_split_clients_module][2]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://nginx.org/en/docs/http/ngx_http_split_clients_module.html "Module ngx_http_split_clients_module"

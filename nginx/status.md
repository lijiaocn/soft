<!-- toc -->
# Nginx 的状态数据

## stub_status

Nginx 的 [stub_status][2] 指令用来显示 nginx 的工作状态，配置方式如下：

```conf
location = /nginx_status {
    allow       127.0.0.1;
    deny        all;

    stub_status  on;    # 开启状态
    access_log  off;    # 不记录该 uri 的访问日志
}
```

效果：

```sh
$ curl 127.0.0.1:9000/nginx_status
Active connections: 1
server accepts handled requests
 3 3 3
Reading: 0 Writing: 1 Waiting: 0
```

[stub_status][2] 给出了各个参数含义：

```conf
Active connections  :活跃连接数
accepts             :接收的连接总数 
handled             :已经处理的连接数  
requests            :客户端请求总数
Reading             :nginx 正在从其中读取 header 的连接数
Writing             :nginx 正在向客户端发送响应数据的连接数
Waiting             :idle 状态的连接
```

同时提供了以下可用变量：

```conf
$connections_active
    same as the Active connections value;
$connections_reading
    same as the Reading value;
$connections_writing
    same as the Writing value;
$connections_waiting
    same as the Waiting value.
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status "stub_status"

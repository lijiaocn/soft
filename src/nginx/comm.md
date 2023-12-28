<!-- toc -->
# Nginx 的常用配置

一些常用的 nginx  配置。

## 日志

设置错误日志文件和日志级别，一般在配置文件最开始处配置：

```conf
error_log  /tmp/logs/error.log  info;
```

日志格式 log_format 分为 [http log_format][2] 和 [stream log_format][3]，两者的格式中可以使用的变量不同。

紧跟在 log_format 后面的字符串是日志格式的名称，nginx 支持定义多个格式，以 http log_format 为例：

```conf
# 格式名称为 main
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
```

http 的日志格式中可以添加任意的 http 头，例如上面的 `$http_user_agent`。

日志格式只用于访问日志，用 access_log 引用引用，access_log 同样分为 [http access_log][4] 和 [stream access_log][5]。http 的 access_log 可以在 location 中单独设置。


使用效果，在 http 中定义 tranproxy：

```conf
http{
    ...省略...
    log_format tranproxy '$remote_addr - $remote_user [$time_local] "$request" '
            '$status $body_bytes_sent "$http_referer" '
            '"$http_user_agent" "$http_x_forwarded_for"';

    ...省略...
}
```

在 location 中引用：

```conf
location / {
    access_log /var/log/nginx/access.80.log tranproxy;
    ... 省略 ...
}
```

访问日志，不带 `-H X-Forwarded-For` 时：

```sh
172.17.0.5 - - [14/Nov/2019:03:12:15 +0000] "GET / HTTP/1.1" 200 467 "-" "curl/7.61.1" "-"
```

访问日志，带有 `-H X-Forwarded-For` 时：

```sh
172.17.0.5 - - [14/Nov/2019:03:12:15 +0000] "GET / HTTP/1.1" 200 467 "-" "curl/7.61.1" "127.0.0.1"
```

## location 的匹配规则

[location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location) 是最常用的，支持四个修饰符： `=`、`~`、` ~*`、`^~`。

* `=`:  严格一致
* `~`:  区分大小写的正则匹配
* `~*`: 不区分大小写的正则匹配
* `^~`: 前缀匹配成功后，忽略正则匹配

正则表达式语法参考 `man 7 regex`，下面的表达式匹配以 /detail 开头且不含有 `.` 路径：

```conf
location ~ '^/detail/[^.]*$ {
    ...省略...
}
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: http://nginx.org/en/docs/http/ngx_http_log_module.html#log_format "http log_format"
[3]: http://nginx.org/en/docs/stream/ngx_stream_log_module.html#log_format "stream log_format"
[4]: http://nginx.org/en/docs/http/ngx_http_log_module.html#access_log "http access_log"
[5]: http://nginx.org/en/docs/stream/ngx_stream_log_module.html#access_log "stream access_log"

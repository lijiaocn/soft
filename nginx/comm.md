<!-- toc -->
# Nginx 的常用配置

一些常用的 nginx  配置。

## 日志

设置错误日志文件和日志级别，一般在配置文件最开始处配置：

```conf
error_log  /tmp/logs/error.log  info;
```

设置日志格式，log_format 在 http 和 stream 中可用，紧跟 log_format 的字符串是日志格式的名称，支持定义多个格式：

```conf
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
```

日志格式指定的是访问日志的格式，访问日志用下面的方式记录：

```conf
access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
access_log off;
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

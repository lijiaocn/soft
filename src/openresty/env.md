<!-- toc -->
# OpenResty 环境准备

## mac 上的安装方法

```sh
brew untap homebrew/nginx
brew install openresty/brew/openresty
```

执行：

```sh
./install-on-mac.sh
```

## 环境验证

准备 openresty 的配置文件，01-hello-world/nginx.conf：

```conf
worker_processes  1;        #nginx worker 数量
error_log logs/error.log;   #指定错误日志文件路径
events {
    worker_connections 1024;
}

http {
    server {
        #监听端口，若你的6699端口已经被占用，则需要修改
        listen 6699;
        location /a {
            default_type text/html;

            content_by_lua_block {
                ngx.say("HelloWorld")
            }
        }
    }
}
```

用该配置文件启动：

```sh
openresty -p `pwd` -c nginx.conf
```

访问：

```sh
$ curl 127.0.0.1:6699/a
HelloWorld
```

## resty 验证

OpenResty 提供一个 resty 命令，可以直接解释执行 lua 脚本，准备一个 lua 脚本 hello.lua：

```lua
#! /usr/bin/env lua
--
-- hello.lua
-- Copyright (C) 2018 lijiaocn <lijiaocn@foxmail.com>
--
-- Distributed under terms of the GPL license.
--
mysql=require "resty.mysql"
ngx.say("hello world")
```

用 resty 执行 lua 脚本：

```sh
$ resty hello.lua
hello world
```


## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

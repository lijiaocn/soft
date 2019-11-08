<!-- toc -->
# OpenResty 中 lua 脚本的使用

OpenResty 用 lua-nginx-module 解释 lua 脚本，lua 语言自身的用法见 [Lua编程速查手册（常用操作)][2]，openresty 提供的 lua 能力（譬如定义的变量、提供的函数等）见 [openresty/lua-nginx-module][3] 。

## OpenResty 执行 Lua 脚本

OpenResty 支持的开发语言是 lua，lua 脚本可以用两种方式运行：

第一种方式，直接写在配置文件中，openresty 加载运行：

```sh
$ cd 01-hello-world
$ openresty -p `pwd` -c nginx.conf
```

这种方式启动了 openresty 服务，访问 openresty 时触发配置中的 lua 脚本的运行：

```sh
$ curl 127.0.0.1:6699/
HelloWorld
```

第二种方式，用 resty 命令直接执行 lua 脚本：

```sh
$ cd 02-hello-world
$ resty hello.lua
hello world
```

## lua_module 的几个执行阶段

lua_module 定义了很多的指令，[lua module directives][5]，其中有一些指令是类似于 server、location 的块指令，它们的作用顺序如下：

![lua_module 指令的的作用顺序](https://cloud.githubusercontent.com/assets/2137369/15272097/77d1c09e-1a37-11e6-97ef-d9767035fc3e.png)

用下面的配置观察这些指令的执行顺序，03-nginx-lua-module/nginx.conf：

```conf
worker_processes  1;             #nginx worker 数量
error_log logs/error.log info;   #指定错误日志文件路径
events {
  worker_connections 256;
}

http {
  server {
    #监听端口，若你的6699端口已经被占用，则需要修改
    listen 6699;
    location / {
        set_by_lua_block $a {
            ngx.log(ngx.INFO, "set_by_lua*")
        }
        rewrite_by_lua_block {
            ngx.log(ngx.INFO, "rewrite_by_lua*")
        }
        access_by_lua_block {
            ngx.log(ngx.INFO, "access_by_lua*")
        }
        content_by_lua_block {
            ngx.log(ngx.INFO, "content_by_lua*")
        }
        header_filter_by_lua_block {
            ngx.log(ngx.INFO, "header_filter_by_lua*")
        }
        body_filter_by_lua_block {
            ngx.log(ngx.INFO, "body_filter_by_lua*")
        }
        log_by_lua_block {
            ngx.log(ngx.INFO, "log_by_lua*")
        }
    }
  }
}
```

执行时打印的日志：

```sh
2019/10/23 15:40:34 [info] 65194#3586531: *1 [lua] set_by_lua:2: set_by_lua*, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1:6699"
2019/10/23 15:40:34 [info] 65194#3586531: *1 [lua] rewrite_by_lua(nginx.conf:17):2: rewrite_by_lua*, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1:6699"
2019/10/23 15:40:34 [info] 65194#3586531: *1 [lua] access_by_lua(nginx.conf:20):2: access_by_lua*, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1:6699"
2019/10/23 15:40:34 [info] 65194#3586531: *1 [lua] content_by_lua(nginx.conf:23):2: content_by_lua*, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1:6699"
2019/10/23 15:40:34 [info] 65194#3586531: *1 [lua] header_filter_by_lua:2: header_filter_by_lua*, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1:6699"
2019/10/23 15:40:34 [info] 65194#3586531: *1 [lua] body_filter_by_lua:2: body_filter_by_lua*, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1:6699"
2019/10/23 15:40:34 [info] 65194#3586531: *1 [lua] log_by_lua(nginx.conf:32):2: log_by_lua* while logging request, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", host: "127.0.0.1:6699"
2019/10/23 15:40:34 [info] 65194#3586531: *1 kevent() reported that client 127.0.0.1 closed keepalive connection
```

## 与 nginx 变量的交互

nginx 有很多 [内置变量][6]，在 lua 脚本中通过 [ngx.var.VARIABLE][7] 获取这些变量。

有一些 nginx 的变量值可以用 lua 修改，大部分是不可以修改的。

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.lijiaocn.com/prog/lua/ "Lua编程速查手册（常用操作)"
[3]: https://github.com/openresty/lua-nginx-module "lua-nginx-module"
[4]: https://github.com/openresty/lua-nginx-module#directives "directives"
[5]: https://github.com/openresty/lua-nginx-module#directives "lua module directives"
[6]: http://nginx.org/en/docs/varindex.html "nginx variables"
[7]: https://github.com/openresty/lua-nginx-module#ngxvarvariable "ngx.var.VARIABLE"

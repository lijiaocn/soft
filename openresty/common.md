<!-- toc -->
# OpenResty 的一些常规操作

OpenResty 提供的开发语言是 Lua，用 lua-nginx-module 解释 lua 脚本，lua 语言自身的用法见 [Lua编程速查手册（常用操作)][2]，openresty 提供的 lua 能力（譬如定义的变量、提供的函数等）见 [openresty/lua-nginx-module][3] 。

## Lua 脚本的两种执行方法

OpenResty 提供的开发语言是 lua，lua 脚本可以用两种方式运行：

第一种方式，直接写在配置文件中，openresty 加载运行：

```sh
$ cd 01-hello-world
$ openresty -p `pwd` -c nginx.conf
```

这种方式启动了 openresty 服务，访问 openresty 服务触发配置中的脚本运行：

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

观察 lua_module 的几个执行阶段，在每个阶段打印日志：

```conf
# 03-nginx-lua-module/nginx.conf
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

会生成下面的日志：

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

## 内存字典

```conf
#04-nginx-lua-module-shared-mem
worker_processes  1;        #nginx worker 数量
error_log logs/error.log;   #指定错误日志文件路径
events {
  worker_connections 1024;
}

 http {
     lua_shared_dict dogs 10m;
     server {
         listen 6699;
         location /set {
             content_by_lua_block {
                 local dogs = ngx.shared.dogs
                 dogs:set("Jim", 8)
                 ngx.say("STORED")
             }
         }
         location /get {
             content_by_lua_block {
                 local dogs = ngx.shared.dogs
                 ngx.say(dogs:get("Jim"))
             }
         }
     }
 }
```

## 变量设置与读取


```conf
#05-nginx-lua-module-readvar
worker_processes  1;        #nginx worker 数量
error_log logs/error.log;   #指定错误日志文件路径
events {
  worker_connections 1024;
}

http {
     server {
         listen 6699;
         location / {
             set $my_var 'my_var';
             content_by_lua_block {
                 local response = {}
                 response[1]= ngx.var.request_uri;
                 response[2]=";"
                 response[3] = ngx.var.my_var;
                 ngx.say(response);
             }
         }
     }
 }
```

## 日志打印

```conf
#06-nginx-lua-module-log
worker_processes  1;              #nginx worker 数量
error_log logs/error.log debug;   #指定错误日志文件路径
events {
  worker_connections 1024;
}

http {
     server {
         listen 6699;
         location / {
             content_by_lua_block {
                ngx.log(ngx.DEBUG,"this is a DEBUG log ",ngx.var.request_uri, ngx.var.host);
                ngx.log(ngx.INFO,"this is a INFO log");
                ngx.log(ngx.NOTICE,"this is a NOTICE log");
                ngx.log(ngx.WARN,"this is a WARN log");
                ngx.log(ngx.ERR,"this is a ERR log");
                ngx.log(ngx.CRIT,"this is a CRIT log");
                ngx.log(ngx.ALERT,"this is a ALERT log");
                ngx.log(ngx.EMERG,"this is a EMERG log");
                ngx.log(ngx.STDERR,"this is a STDERR log");
             }
         }
     }
 }
```

## 重定向

```conf
#study-OpenResty/example/07-nginx-lua-module-redirect
worker_processes  1;              #nginx worker 数量
error_log logs/error.log debug;   #指定错误日志文件路径
events {
    worker_connections 1024;
}

http {
    server {
        listen 6699;
        location / {
            access_by_lua_block{
                local host = ngx.var.host
                -- 不匹配
                local replace,n,err  = ngx.re.sub(ngx.var.request_uri, "/abc", "123")
                ngx.log(ngx.DEBUG, "uri is: ", ngx.var.request_uri, " ,replace is: ",replace, " ,n is: ",n , " ,err is: ",err)
                ngx.redirect(replace,301)
            }
        }
    }
}
```

## 4 层负载均衡

```conf
#study-OpenResty/example/08-nginx-lua-module-balancer
worker_processes  1;              #nginx worker 数量
error_log logs/error.log debug;   #指定错误日志文件路径
events {
    worker_connections 256;
}

stream{
        upstream upstream_balancer{
                server 0.0.0.1:6699; # placeholder
                balancer_by_lua_block {
                   local ngx_balancer = require("ngx.balancer")
                   ngx_balancer.set_more_tries(1)
                   local ok, err = ngx_balancer.set_current_peer("127.0.0.1","9090")
                   if not ok then
                      ngx.log(ngx.ERR, string.format("error while setting current upstream : %s", err))
                   end
                }
        }
        server {
                proxy_timeout           600s;
                listen                  6699;
                proxy_pass              upstream_balancer;
        }
}
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.lijiaocn.com/prog/lua/ "Lua编程速查手册（常用操作)"
[3]: https://github.com/openresty/lua-nginx-module "lua-nginx-module"
[4]: https://github.com/openresty/lua-nginx-module#directives "directives"

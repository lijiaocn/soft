<!-- toc -->
# OpenResty 的一些常规操作

## 内存字典

04-nginx-lua-module-shared-mem/nginx.conf：

```conf
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

05-nginx-lua-module-readvar/nginx.conf：

```conf
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

06-nginx-lua-module-log/nginx.conf：

```conf
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

07-nginx-lua-module-redirect/nginx.conf：

```conf
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

08-nginx-lua-module-balancer/nginx.conf：

```conf
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

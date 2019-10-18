<!-- toc -->
# Nginx 配置文件格式

Nginx 的源代码中有个配置文件 [nginx.conf](https://github.com/nginx/nginx/blob/master/conf/nginx.conf) 样例，CentOS 安装的 nginx 自带的默认配置文件结构更好。

## CentOS 的 nginx 配置文件组织方式

> mac 上的 nginx 的配置文件组织方式和 CentOS 类似，配置文件位于 /usr/local/etc/nginx/nginx.conf

用 yum 命令在 CentOS 中安装 nginx 的，默认使用的配置文件是 `/etc/nginx/nginx.conf`：

```ini
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf; 

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    # 默认的server
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # 默认server的配置也写到单独的配置文件中
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# TLS 配置方法
# Settings for a TLS enabled server.
    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/pki/nginx/server.crt";
        ssl_certificate_key "/etc/pki/nginx/private/server.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

上面的配置文件中有两个 `include` 指令需要特别注意，这两个指令又引入了其它的配置文件，它们分别加载了 Nginx 模块和 Server 配置。

### 第一个 include：加载 nginx 模块

第一个 include 指令是导入 nginx 模块的加载指令：

```conf
# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf; 
```

在 /usr/share/nginx/modules 目录中有多个配置文件，每个配置文件用来加载一个 nginx 模块：

```bash
$ ls  /usr/share/nginx/modules/
mod-http-geoip.conf  mod-http-image-filter.conf  mod-http-perl.conf  
mod-http-xslt-filter.conf  mod-mail.conf  mod-stream.conf
```

以 `mod-http-geoip.conf` 为例，模块的加载命令如下：

```conf
load_module "/usr/lib64/nginx/modules/ngx_http_geoip_module.so";
```

### 第一个 include：加载 server 配置

第二个 include 加载 server 配置：

```conf
include /etc/nginx/conf.d/*.conf;
```

Nginx 是一个代理服务器，它可以同时代理多个后端服务，将这些后端服务的配置写到各自的单独的配置文件中，可以简化配置文件并且方便管理。

比如说 /etc/nginx/conf.d/flarum.conf 中全都是 flarum.local 服务相关的配置：

```conf
server {
    listen       80 ;
    listen       [::]:80 ;
    server_name  flarum.local;                         # 在本地 host 配置域名
    root         /vagrant/flarum/2_flarum/project;     # 这里是 composer 安装的 flarum 项目目录

    location / { try_files $uri $uri/ /index.php?$query_string; }
    location /api { try_files $uri $uri/ /api.php?$query_string; }
    location /admin { try_files $uri $uri/ /admin.php?$query_string; }

    location /flarum {
            deny all;
            return 404;
    }

    location ~* \.php$ {
            fastcgi_split_path_info ^(.+.php)(/.+)$;
            fastcgi_pass 127.0.0.1:9000;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param HTTP_PROXY ""; # Fix for https://httpoxy.org/ vulnerability
            fastcgi_index index.php;
    }
    #...省略部分内容...
    gzip on;
    gzip_http_version 1.1;
    gzip_vary on;
}
```

## API 网关 kong 的配置文件组织方式

Kong 是基于 OpenResty 实现的 API 网关，OpenResty 基于 nginx，kong 使用的 nginx.conf 如下：

```conf
worker_processes auto;
daemon off;

pid pids/nginx.pid;
error_log logs/error.log debug;

worker_rlimit_nofile 1024;

events {
    worker_connections 1024;
    multi_accept on;
}

http {
    # kong的配置单独写入到另一个配置文件中
    include 'nginx-kong.conf';
}
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

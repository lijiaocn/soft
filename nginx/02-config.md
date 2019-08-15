<!-- toc -->
# Nginx配置文件格式

nginx的源代码中有个配置文件[nginx.conf](https://github.com/nginx/nginx/blob/master/conf/nginx.conf)样例，但是CentOS安装的nginx自带的默认配置文件结构更好。

## CentOS中Nginx配置文件

用yum命令在CentOS中安装nginx的，默认使用的配置文件是`/etc/nginx/nginx.conf`：

```conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# 为每个要加载的nginx模块单独创建一个配置文件
# 例如/usr/share/ngxin/modules/mod-http-geoip.conf的内容如下：
#
#    load_module "/usr/lib64/nginx/modules/ngx_http_geoip_module.so";
#
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

# TLS配置方法
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

上面的配置文件中有两个`include`指令需要特别注意，这两个指令又引入了其它的配置文件，它们分别加载了Nginx模块和Server配置。

### 加载Nginx模块

第一个include指令是导入nginx模块的加载指令：

```conf
# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf; 
```

在/usr/share/nginx/modules目录中有多个配置文件，每个配置文件用来加载一个nginx模块：

```bash
$ ls  /usr/share/nginx/modules/
mod-http-geoip.conf  mod-http-image-filter.conf  mod-http-perl.conf  
mod-http-xslt-filter.conf  mod-mail.conf  mod-stream.conf
```

以`mod-http-geoip.conf`为例，加载模块的配置文件内容格式如下：

```conf
load_module "/usr/lib64/nginx/modules/ngx_http_geoip_module.so";
```

### 加载Server配置

第二个include是加载server的配置：

```conf
include /etc/nginx/conf.d/*.conf;
```

Nginx是一个代理服务器，它可以同时代理多个后端服务，将这些后端服务的配置写到各自的单独的配置文件中，极大的简化了配置文件，并且方便管理。

比如说/etc/nginx/conf.d/flarum.conf中全都是flarum.local服务相关的配置：

```conf
server {
    listen       80 ;
    listen       [::]:80 ;
    server_name  flarum.local;                         # 在本地host配置域名
    root         /vagrant/flarum/2_flarum/project;     # 这里是composer安装的flarum项目目录

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

### API网关kong的配置文件组织方式

可以用更大的粒度拆分配置，例如kong使用的nginx.conf：

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

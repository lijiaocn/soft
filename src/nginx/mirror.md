# Nginx 请求复制功能

Nginx 1.13.4 引入了 [ngx_http_mirror_module][2]，实现了 http 请求复制的功能。

## 准备接收复制请求的服务

启动两个容器一个正常处理请求，一个接收复制到请求：

```sh
docker rm -f echoserver
docker run -idt --name echoserver -p 9090:8080 googlecontainer/echoserver:1.10

docker rm -f http-record
docker run -idt --name http-record -p 9091:8080 lijiaocn/http-record:0.0.1
```

执行：

```sh
./start_backends.sh
```

## 配置请求复制

在 nginx 的配置文件中，配置镜像复制，参考 [ngx_http_mirror_module][2]。

```conf
upstream echo_upstream{
    server  127.0.0.1:9090;
    keepalive 1;
}

upstream http-record_upstream{
    server  127.0.0.1:9091;
    keepalive 1;
}

server {
    listen       9000;
    listen       [::]:9000;
    server_name  echo.example;
    keepalive_requests  2000;
    keepalive_timeout 60s;

    location / {
        mirror /mirror;
        proxy_pass  http://echo_upstream;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }

    location = /mirror {
        internal;
        proxy_pass http://http-record_upstream$request_uri;
    }
}
```

所有发送到 echo.example 的请求将被复制一份到 http-record，请求端收到的是 echo_upstream 的回应，http-record_upstream 的回应将被忽略。

## 复制效果

发起请求，收到的是 echo 的回应：

```sh
$ curl -H "Host: echo.example" 127.0.0.1:9000/abcd/eft

Hostname: 57e34b409aa1

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.1
	method=GET
	real path=/abcd/eft
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://echo_upstream:8080/abcd/eft

Request Headers:
	accept=*/*
	host=echo_upstream
	user-agent=curl/7.54.0

Request Body:
	-no body in request-
```

查看 http-record 容器的日志，会发现收到了完全相同的请求：

```sh
/go/src/Server/echo.go:46: {
    "RemoteAddr": "172.17.0.1:46856",
    "Method": "GET",
    "Host": "http-record_upstream",
    "RequestURI": "/abcd/eft",
    "Header": {
        "Accept": [
            "*/*"
        ],
        "Connection": [
            "close"
        ],
        "User-Agent": [
            "curl/7.54.0"
        ]
    },
    "Body": ""
}
```

## 配置修正

仔细观察上面的配置 Host 是 http-record_upstream，不是原始的 host。将配置修正为：

```conf
    location / {
        access_log /tmp/logs/access.log mirror;
        mirror /mirror;

        # 添加这行配置
        set $proxy_host $host;    

        proxy_http_version 1.1;
        proxy_set_header Connection "";

        proxy_pass  http://echo_upstream;
    }
```

## 结合 split_clients 实现请求的部分复制

用于 A/B 测试的 [split_clients](./abtest.md) 指令和 mirror 指令配合，可以实现请求的按比例复制，split_clients 的用法见 [Nginx的A/B测试功能](./abtest.md)。

将 50% 的流量复制到复制到 record_upstream： 

```conf
split_clients "$date_gmt" $mirror_servers {
    50%        record_upstream;
    *          "";
}

server {
    listen       9000;
    server_name  echo.example;
    keepalive_requests  1000;
    keepalive_timeout 60s;

    location / {
        access_log /tmp/nginx_access.log mirror;
        mirror /mirror;

        proxy_pass  http://echo_upstream;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }

    location = /mirror {
        internal;
        access_log /tmp/nginx_mirror_access.log mirror;
        if ($mirror_servers) {
            proxy_pass http://$mirror_servers$request_uri;
        }
    }
}
```

mirror_servers 也可以是域名，域名需要指定 dns nameserver：

```sh
...省略...
split_clients "$date_gmt" $mirror_servers {
#   50%        record_upstream;
    100%       local.lijiaocn.com:9091;
    *          "";
}
...省略...
    location = /mirror {
        internal;
        access_log off;
        #access_log /tmp/nginx_mirror_access.log mirror;
        resolver 114.114.114.114;

        if ($mirror_servers) {
            set $proxy_host $host;
            proxy_pass http://$mirror_servers$request_uri;
        }
    }
...省略...
```

## 参考

1. [李佶澳的博客][1]
2. [ngx_http_mirror_module][2]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://nginx.org/en/docs/http/ngx_http_mirror_module.html" "ngx_http_mirror_module"

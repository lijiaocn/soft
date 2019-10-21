# Nginx 学习使用的试验环境

在 mac 上安装 nginx :

```sh
$ brew install nginx
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you do nott want/need a background service you can just run:
  nginx
==> Summary
🍺  /usr/local/Cellar/nginx/1.17.3_1: 25 files, 2MB
```

mac 上的 nginx 的配置文件为：/usr/local/etc/nginx/nginx.conf

server 配置的放置目录：/usr/local/etc/nginx/servers/

## 验证 nginx

启动：

```sh
$ brew services start nginx
```

查看状态：

```sh
$ brew services list
Name       Status  User   Plist
nginx      started lijiao /Users/lijiao/Library/LaunchAgents/homebrew.mxcl.nginx.plist
```

访问：

```sh
$ curl 127.0.0.1:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

停止：

```sh
$ brew services stop nginx
```

## 验证代理功能

在本地启动一个 [echoserver](../envoy/echoserver.md) 服务：

```sh
docker run -idt --name echoserver -p 9090:8080 -p 9443:8443 googlecontainer/echoserver:1.10
```

执行：

```sh
./start_echoserver.sh
```

创建配置文件，echo.example.conf：

```conf
upstream echo_upstream{
    server  127.0.0.1:9090;
    keepalive 1;
}

server {
    listen       9000;
    listen       [::]:9000;
    server_name  echo.example;
    keepalive_requests  2000;
    keepalive_timeout 60s;

    location / {
        proxy_pass  http://echo_upstream;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

连接到 /usr/local/etc/nginx/servers/ 目录中：

```sh
./link.sh echo.example.conf
```

验证配置：

```sh
/usr/local/bin/nginx -c /usr/local/etc/nginx/nginx.conf
```

重启 nginx 后，访问：

```sh
$ curl -H "Host: echo.example" 127.0.0.1:9000

Hostname: 12a852d64212

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

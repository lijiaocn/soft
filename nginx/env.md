## Nginx 学习用到的试验环境

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

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

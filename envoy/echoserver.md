# 用 echoserver 观察代理/转发效果

echoserver 是一个回显请求的 http 服务，用来观察 http 请求的代理/转发效果非常方便。

下载镜像：

```sh
docker pull googlecontainer/echoserver:1.10
```

启动：

```sh
$ docker run -idt  -p 8080:8080 -p 8443:8443 googlecontainer/echoserver:1.10
```

访问：

```sh
$ curl 127.0.0.1:8080
Hostname: 611185215d7a

Pod Information:
    -no pod information available-

Server values:
    server_version=nginx: 1.13.3 - lua: 10008

Request Information:
    client_address=172.17.0.1
    method=GET
    real path=/
    query=
    request_version=1.1
    request_scheme=http
    request_uri=http://127.0.0.1:8080/

Request Headers:
    accept=*/*
    host=127.0.0.1:8080
    user-agent=curl/7.54.0

Request Body:
    -no body in request-
```

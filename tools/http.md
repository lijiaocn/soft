<!-- toc -->
# HTTP 协议相关的工具

## HTTP 请求回显: echoserver

下载镜像：

```sh
docker pull googlecontainer/echoserver:1.10 
```

启动：

```sh
$ docker run -idt --name echoserver -p 9090:8080 -p 8443:8443 googlecontainer/echoserver:1.10
```

直接访问 echo 容器效果如下：

```sh
$ curl 127.0.0.1:9090
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
    request_uri=http://127.0.0.1:9090/

Request Headers:
    accept=*/*
    host=127.0.0.1:9090
    user-agent=curl/7.54.0

Request Body:
    -no body in request-
```

## HTTP 请求记录: http-record

echoserver 向客户端返回接收的请求情况，http-record 不仅向客户端返回，同时在本地的标准输出打印日志。在测试流量复制功能时，复制的请求的回应会被丢弃，可以用 http-record 观察请求是否被复制。

```sh
docker run -idt --name http-record -p 9091:8080 lijiaocn/http-record:0.0.1
```

请求：

```sh
$ curl 127.0.0.1:9091
{
    "RemoteAddr": "172.17.0.1:49802",
    "Method": "GET",
    "Host": "127.0.0.1:9091",
    "RequestURI": "/",
    "Header": {
        "Accept": [
            "*/*"
        ],
        "User-Agent": [
            "curl/7.54.0"
        ]
    },
    "Body": ""
}%
```

容器日志：

```sh
$ docker logs -f http-record
/go/src/Server/echo.go:46: {
    "RemoteAddr": "172.17.0.1:49802",
    "Method": "GET",
    "Host": "127.0.0.1:9091",
    "RequestURI": "/",
    "Header": {
        "Accept": [
            "*/*"
        ],
        "User-Agent": [
            "curl/7.54.0"
        ]
    },
    "Body": ""
}
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

<!-- toc -->
# ingress-nginx 的常用注解

[Annotations][2] 中列出了 ingress-nginx 支持的 annotation，

## 后端超时设置

```sh
nginx.ingress.kubernetes.io/proxy-connect-timeout    # 后端连接超时，默认 60 秒
nginx.ingress.kubernetes.io/proxy-send-timeout       # 后端发送超时，默认 120 秒
nginx.ingress.kubernetes.io/proxy-read-timeout       # 后端读取超时，默认 120 秒
```

## 后端重试设置

重试条件设置：

```sh
nginx.ingress.kubernetes.io/proxy-next-upstream  # 默认 "error timeout"
```
proxy-next-upstream 是 nginx 的 [标准指令][3]，它定义了将请求转发另一个 upstream 的情况，支持以下的值，可以同时使用：

	error 
	timeout 
	invalid_header 
	http_500 
	http_502 
	http_503 
	http_504 
	http_403 
	http_404 
	http_429 
	non_idempotent 
	off

重试超时设置：

```sh
nginx.ingress.kubernetes.io/proxy-next-upstream-timeout   # 超时时间，默认 0 秒
nginx.ingress.kubernetes.io/proxy-next-upstream-tries     # 重试次数，默认 3 次 
```

请求缓存：

```sh
nginx.ingress.kubernetes.io/proxy-request-buffering       # 缓存请求，默认 on
```

开启请求缓存时，ingress 暂存收到的客户端的请求，请求数据全部收齐后，再将请求转发给 upstream。

## 缓存区设置

请求缓冲区大小，默认为两个内存页面，通常是 8k 和 16k，超过的请求被写入临时文件暂存：

```sh
nginx.ingress.kubernetes.io/client-body-buffer-size  "1000" # 默认 8k
```

```sh
nginx.ingress.kubernetes.io/proxy-body-size
```

客户端请求最大长度：

```sh
nginx.ingress.kubernetes.io/proxy-body-size  # 默认 8m
```

响应数据缓冲区大小（从响应头读取的第一块数据大小），默认为 1 个页面，通常是 4k 和 8k：

```sh
nginx.ingress.kubernetes.io/proxy-buffer-size # 默认 4k
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/ "Nginx Ingress Controller Annotations"
[3]: http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream "proxy_next_upstream"

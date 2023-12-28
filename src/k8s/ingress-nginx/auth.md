<!-- toc -->
# ingress-nginx 的认证功能使用示例

## Basic Authentication （HTTP 认证）

[Basic Authentication][2] 是最简单的 http 认证方式，采用用户名和密码的方式，用户名和密码以 secret 的方式存放在 kubernetes 中。

```sh
cd 02-auth-basic
```

### 创建用户，设置密码

用下面的三个操作，创建 basic-auth 用户 foo，密码 123456，将用户信息提交到 kubernetes：

```sh
$ htpasswd -c auth foo
$ kubectl -n demo-echo create secret generic basic-auth --from-file=auth

```

secret 与目标服务 echo 在同一个 namespace 中。

执行效果如下：

```sh
$ ./01-create-user.sh
New password:  123456
Re-type new password: 123456
Adding password for user foo
...
```

### 为目标服务设置 ingress

为目标服务创建一个启用了 basic-auth 的 ingress：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-auth-basic
  annotations:
    # type of authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    # name of the secret that contains the user/password definitions
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # message to display with an appropriate context why the authentication is required
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'
spec:
  rules:
  - host: auth-basic.echo.example
    http:
      paths:
      - path: /
        backend:
          serviceName: echo
          servicePort: 80
```

执行：

```sh
$ kubectl -n demo-echo apply -f auth-basic-ingress.yaml
```

```sh
$ kubectl -n demo-echo get ingress
NAME                           HOSTS                     ADDRESS   PORTS   AGE
ingress-echo                   echo.example                        80      42d
ingress-echo-with-auth-basic   auth-basic.echo.example             80      6s
```

### 使用效果

通过开启了认证功能的 ingress 访问，不带用户名和密码时，返回 401：

```sh
$ curl -H "Host: auth-basic.echo.example" 192.168.99.100:30933
<html>
<head><title>401 Authorization Required</title></head>
<body>
<center><h1>401 Authorization Required</h1></center>
<hr><center>openresty/1.15.8.1</center>
</body>
</html>
```

用户名或密码错误时：

```sh
$ curl -H "Host: auth-basic.echo.example" 192.168.99.100:30933 -u 'foo:bar'
<html>
<head><title>401 Authorization Required</title></head>
<body>
<center><h1>401 Authorization Required</h1></center>
<hr><center>openresty/1.15.8.1</center>
</body>
```

用户名和密码正确时：

```sh
$ curl -H "Host: auth-basic.echo.example" 192.168.99.100:30933 -u 'foo:123456'

Hostname: echo-597d89dcd9-4dp6f

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008
```

## Client Certificate Authentication（客户端证书认证）

[Client Certificate Authentication][2]，进行客户端证书认证需要提供 **客户端证书** 、 **服务端证书** 和能够验证客户端证书的 **ca 证书**。启用客户端证书必须使用 tls 加密，所以服务端证书是必须的。

操作过程中，遇到了两个低级错误引发的问题，供参考：

* [CONNECT_CR_SRVR_HELLO:wrong version number][4]
* [unable to get local issuer certificate][5]

```sh
cd 03-auth-cert
```

### 生成证书

用下面的命令生成证书：

```sh

echo "生成自签署的 ca 证书"
openssl req -x509 -sha256 -newkey rsa:4096 -keyout ca.key -out ca.crt -days 3560 -nodes -subj '/CN=My Cert Authority'

echo "生成用上述 ca 签署的 server 证书"
openssl req -new -newkey rsa:4096 -keyout server.key -out server.csr -nodes -subj '/CN=auth-cert.echo.example'
openssl x509 -req -sha256 -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt

echo "生成用上述 ca 签署的 client 证书"
openssl req -new -newkey rsa:4096 -keyout client.key -out client.csr -nodes -subj '/CN=My Client'
openssl x509 -req -sha256 -days 3650 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 02 -out client.crt
```

执行：

```sh
$ ./01-create-cert.sh
```

生成下面的文件：

```sh
client.crt        server.crt
ca.crt            client.csr        server.csr
ca.key            client.key        server.key
```

### 上传证书

将 ca 证书和服务端证书上传到目标服务 echo 所在的 namespace：

```sh
$ kubectl -n demo-echo create secret generic ca-secret --from-file=ca.crt=ca.crt
$ kubectl -n demo-echo create secret generic tls-secret --from-file=tls.crt=server.crt --from-file=tls.key=server.key
```

### 创建对应的 ingress

创建配置了 tls 认证的 ingress，auth-tls-secret 是服务端验证客户端证书时使用的证书：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-echo-with-auth-cert
  annotations:
    # Enable client certificate authentication
    nginx.ingress.kubernetes.io/auth-tls-verify-client: "on"
    # Create the secret containing the trusted ca certificates
    nginx.ingress.kubernetes.io/auth-tls-secret: "demo-echo/ca-secret"
    # Specify the verification depth in the client certificates chain
    nginx.ingress.kubernetes.io/auth-tls-verify-depth: "1"
    # Specify an error page to be redirected to verification errors
    nginx.ingress.kubernetes.io/auth-tls-error-page: "http://auth-cert.echo.example/error-cert.html"
    # Specify if certificates are passed to upstream server
    nginx.ingress.kubernetes.io/auth-tls-pass-certificate-to-upstream: "true"
spec:
  rules:
  - host: auth-cert.echo.example
    http:
      paths:
      - path: /
        backend:
          serviceName: echo
          servicePort: 80
  tls:
  - hosts:
    - auth-cert.echo.example
    secretName: tls-secret
```

执行：

```sh
$ kubectl -n demo-echo apply -f auth-cert-ingress.yaml
```

```sh
$ kubectl -n demo-echo get ingress
NAME                           HOSTS                     ADDRESS   PORTS   AGE
ingress-echo                   echo.example                        80      42d
ingress-echo-with-auth-basic   auth-basic.echo.example             80      3s
ingress-echo-with-auth-cert    auth-cert.echo.example              80      81s
```

### 使用效果

使用 http 协议访问时，308 跳转到 https 网址：

```sh
$ curl -v  -H "Host: auth-cert.echo.example" 192.168.99.100:30933
* Rebuilt URL to: 192.168.99.100:30933/
*   Trying 192.168.99.100...
* TCP_NODELAY set
* Connected to 192.168.99.100 (192.168.99.100) port 30933 (#0)
> GET / HTTP/1.1
> Host: auth-cert.echo.example
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 308 Permanent Redirect
< Server: openresty/1.15.8.1
< Date: Fri, 11 Oct 2019 11:34:57 GMT
< Content-Type: text/html
< Content-Length: 177
< Connection: keep-alive
< Location: https://auth-cert.echo.example/
```

使用 https 协议访问（使用 https 时必须使用域名），客户端没有使用证书时，被重定向：

```sh
$ curl --cacert  ca.crt  https://auth-cert.echo.example:30358/
...
< HTTP/2 302
< server: openresty/1.15.8.1
< date: Sat, 12 Oct 2019 06:44:11 GMT
< content-type: text/html
< content-length: 151
< location: http://auth-cert.echo.example/error-cert.html
<
...
<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>openresty/1.15.8.1</center>
</body>
</html>
```

重定向地址在 annotation 中设置：

```sh
nginx.ingress.kubernetes.io/auth-tls-error-page: "http://auth-cert.echo.example/error-cert.html"
```

使用客户端证书认证：

```sh
$ curl --cacert  ca.crt --cert  client.crt --key client.key  https://auth-cert.echo.example:30358/

Hostname: echo-597d89dcd9-4dp6f

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008
...
```

## 参考

1. [李佶澳的博客][1]
2. [Basic Authentication][6]
3. [Client Certificate Authentication][7]
4. [CONNECT_CR_SRVR_HELLO:wrong version number][4]
5. [unable to get local issuer certificate][5]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.github.io/ingress-nginx/examples/auth/basic/ "Basic Authentication"
[3]: https://kubernetes.github.io/ingress-nginx/examples/auth/client-certs/ "Client Certificate Authentication"
[4]: https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2019/10/12/ssl-wrong-version-number.html "CONNECT_CR_SRVR_HELLO:wrong version number"
[5]: https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2019/10/12/cacert-not-work-on-mac.html "unable to get local issuer certificate"
[6]: https://kubernetes.github.io/ingress-nginx/examples/auth/basic/ "Basic Authentication"
[7]: https://kubernetes.github.io/ingress-nginx/examples/auth/client-certs/ "Client Certificate Authentication"

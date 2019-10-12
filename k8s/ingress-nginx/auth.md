<!-- toc -->
# ingress-nginx 的认证功能使用示例

## Basic Authentication （HTTP 认证）

[Basic Authentication][2] 是最简单的 http 认证方式，采用用户名和密码的方式，用户名和密码以 secret 的方式存放在 kubernetes 中。

### 创建用户，设置密码

用下面的三个操作，创建 basic-auth 用户 foo，密码 123456：

```sh
htpasswd -c auth foo

kubectl -n demo-echo create secret generic basic-auth --from-file=auth

kubectl -n demo-echo get secret basic-auth -o yaml
```

secret 与目标服务 echo 在同一个 namespace 中。

执行效果如下：

```sh
$ ./01-create-user.sh
New password:  123456
Re-type new password: 123456
Adding password for user foo
secret/basic-auth created
apiVersion: v1
data:
  auth: Zm9vOiRhcHIxJHpPbzBUY3E2JHZBa3hQRzZMYWdaLjVqdzRFR1RISzEK
kind: Secret
metadata:
  creationTimestamp: "2019-10-11T09:18:13Z"
  name: basic-auth
  namespace: demo-echo
  resourceVersion: "729951"
  selfLink: /api/v1/namespaces/demo-echo/secrets/basic-auth
  uid: 5d557200-3cbc-4fc6-8965-f0f8ac3a7ade
type: Opaque
```

### 为目标服务设置 ingress

为目标服务创建一个启用了 basic-auth 的 ingress：

```sh
$ kubectl -n demo-echo apply -f auth-basic-ingress.yaml
ingress.extensions/ingress-with-auth created

$ kubectl -n demo-echo get ingress
NAME                           HOSTS                     ADDRESS   PORTS   AGE
ingress-echo                   echo.example                        80      42d
ingress-echo-with-auth-basic   auth-basic.echo.example             80      6s
```

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

[Client Certificate Authentication][2]，进行客户端证书认证需要提供 **客户端证书** 、 **服务端证书** 和能够验证客户端证书的 **ca 证书**。

启用客户端证书必须使用 tls 加密，所以服务端证书是必须的。

操作过程中，遇到了两个低级错误引发的问题，供参考：

* [CONNECT_CR_SRVR_HELLO:wrong version number][4]
* [unable to get local issuer certificate][5]

### 生成证书

用下面的命令生成证书：

```sh
echo "生成 ca 证书"
openssl req -x509 -sha256 -newkey rsa:4096 -keyout ca.key -out ca.crt -days 3560 -nodes -subj '/CN=My Cert Authority'

echo "生成用上述 ca 签署的 server 证书"
openssl req -new -newkey rsa:4096 -keyout server.key -out server.csr -nodes -subj '/CN=mydomain.com'
openssl x509 -req -sha256 -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt

echo "生成用上述 ca 签署的 client 证书"
openssl req -new -newkey rsa:4096 -keyout client.key -out client.csr -nodes -subj '/CN=My Client'
openssl x509 -req -sha256 -days 3650 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 02 -out client.crt
```

执行过程如下：

```sh
$ ./01-create-cert.sh
生成自签署的 ca 证书
Generating a 4096 bit RSA private key
.................................................................++
...............++
writing new private key to 'ca.key'
-----
生成用上述 ca 签署的 server 证书
Generating a 4096 bit RSA private key
..........................................................++
..........................................................++
writing new private key to 'server.key'
-----
Signature ok
subject=CN = mydomain.com
Getting CA Private Key
生成用上述 ca 签署的 client 证书
Generating a 4096 bit RSA private key
.........................................++
..........................................................++
writing new private key to 'client.key'
-----
Signature ok
subject=CN = My Client
Getting CA Private Key
```

生成了下面的文件：

```sh
client.crt        server.crt
ca.crt            client.csr        server.csr
ca.key            client.key        server.key
```

### 上传证书

将证书上传到目标服务 echo 所在的 namespace：

```sh
$ kubectl -n demo-echo create secret generic ca-secret --from-file=ca.crt=ca.crt
$ kubectl -n demo-echo create secret generic tls-secret --from-file=tls.crt=server.crt --from-file=tls.key=server.key
```

### 创建对应的 ingress

```sh
$ kubectl -n demo-echo apply -f auth-cert-ingress.yaml
ingress.extensions/ingress-echo-with-auth-cert created

$ kubectl -n demo-echo get ingress
NAME                           HOSTS                     ADDRESS   PORTS   AGE
ingress-echo                   echo.example                        80      42d
ingress-echo-with-auth-basic   auth-basic.echo.example             80      3s
ingress-echo-with-auth-cert    auth-cert.echo.example              80      81s
```

### 认证效果

使用 http 协议访问，308 跳转到 https 网址：

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

使用 https 协议访问，注意使用 https 时必须使用域名（在本地 /etc/hosts 中配置），客户端没有使用证书时，被重定向：

```sh
$ curl --cacert  ca.crt   https://auth-cert.echo.example:30358/
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
2. [CONNECT_CR_SRVR_HELLO:wrong version number][4]
3. [unable to get local issuer certificate][5]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.github.io/ingress-nginx/examples/auth/basic/ "Basic Authentication"
[3]: https://kubernetes.github.io/ingress-nginx/examples/auth/client-certs/ "Client Certificate Authentication"
[4]: https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2019/10/12/ssl-wrong-version-number.html "CONNECT_CR_SRVR_HELLO:wrong version number"
[5]: https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2019/10/12/cacert-not-work-on-mac.html "unable to get local issuer certificate"

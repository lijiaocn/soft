<!-- toc -->
# istio 的基本概念：ServiceEntry

[ServiceEntry][1] 用来将外部的服务封装成 istio 网格中的服务，为网格内和网格外的服务的统一管理提供基础，详情见 [Service Entry Detail][2]。

外部服务被封装为 ServiceEntry，可以像 kubernetes 内部的服务一样使用。

## 封装外部的域名

外部服务是域名或 IP，下面是域名的例子：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-https
spec:
  hosts:
  - api.dropboxapi.com
  - www.googleapis.com
  - api.facebook.com
  location: MESH_EXTERNAL
  ports:
  - number: 443
    name: https
    protocol: TLS
  resolution: DNS
```

hosts 是外部服务在网格内的名称，它可以与外部服务的域名相同，可以不同，取决网格内要使用的域名。上面的例子，没有配置外部服务的原始域名，默认原始域名与 hosts 中的域名相同。

如果外部服务在网格内的域名和网格外的域名不同，在 endpoints 中配置原始域名：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-dns
spec:
  hosts:
  - foo.bar.com
  location: MESH_EXTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
  endpoints:
  - address: us.foo.bar.com
    ports:
      https: 8080
  - address: uk.foo.bar.com
    ports:
      https: 9080
  - address: in.foo.bar.com
    ports:
      https: 7080
```

resolution 指定解析方式为 DNS。

## 封装 unix domain socket

unix domain socket 地址也可以封装到网格内：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: unix-domain-socket-example
spec:
  hosts:
  - "example.unix.local"
  location: MESH_EXTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: STATIC
  endpoints:
  - address: unix:///var/run/example/socket
```

## 为外部服务配置 vip

将外部服务的多个 ip 封装成名为 mymongodb.somedomain 的网格内服务，并设置网格内 vip：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-mongocluster
spec:
  hosts:
  - mymongodb.somedomain # not used
  addresses:
  - 192.192.192.192/24 # VIPs
  ports:
  - number: 27018
    name: mongodb
    protocol: MONGO
  location: MESH_INTERNAL
  resolution: STATIC
  endpoints:
  - address: 2.2.2.2
  - address: 3.3.3.3
```

## 设置外部服务的转发策略

外部服务转发策略的设置方法和内部服务相同，用 [DestinationRule](./dstrule.md) 设置，下例子中的 mymongodb.somedomain 是外部服务在网格内的名称：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: mtls-mongocluster
spec:
  host: mymongodb.somedomain
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /etc/certs/myclientcert.pem
      privateKey: /etc/certs/client_private_key.pem
      caCertificates: /etc/certs/rootcacerts.pem
```

## 参考

[1]: https://istio.io/docs/concepts/traffic-management/#service-entries "Service Entry"
[2]: https://istio.io/docs/reference/config/networking/v1alpha3/service-entry/ "Service Entry Detail"

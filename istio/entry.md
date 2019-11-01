<!-- toc -->
# istio 的基本概念：Service Entry

[Service Entry][1] 用来将外部的服务封装成 istio 网格中的服务，为统一管理网格内和网格外的服务提供基础，详情见 [Service Entry Detail][2]。

外部的服务被封装为 Service Entry 之后，可以像 kubernetes 内部的服务一样引用。

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

hosts 是外部服务在网格内的名称，它可以是正好是外部服务的域名，也可以不是。上面的例子中 resolution 指定解析方式为 DNS，外部服务的域名和它们在网格内的名称一致。

可以为外部服务任意设置一个网格内的名称，然后在 endpoints 中配置外部服务的外部地址：

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

将外服服务的多个 ip 地址封装成名为 mymongodb.somedomain 的网格内服务，并设置网格内 vip：

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

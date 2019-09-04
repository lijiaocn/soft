<!-- toc -->
# Service Entry

[Service Entry][1] 用来将外部的服务引入到 istio 的服务网格中，详情见 [Service Entry Detail][2]。

将外部的服务封装为 Service Entry 之后，可以和 kubernetes 内部的服务统一管理。

## 封装外部的域名

外部服务地址可以是域名，hosts 就是外部的域名：

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

可以采用多个不同的域名作为 endpoint：

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

unix domain socket 地址也可以封装：

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

## 封装外部的 IP

将外服服务的 IP 地址封装到名为 mymongodb.somedomain 的服务，并设置 VIP：

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

## 设置外部服务的负载均衡策略

外部服务的负载均衡的策略同样用 [DestinationRule](./dstrule.md) 设置，host 为封装后的服务名称：

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

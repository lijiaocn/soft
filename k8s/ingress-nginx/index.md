# ingress-nginx 的使用方法

[ingress-nginx][3] 是 NGINX Ingress Controller for Kubernetes，它能做的事情越来越多，发展比较快。[NGINX Ingress Controller Examples][2] 列出了 ingress-nginx 能做的事情。这里逐个尝试一下。

## 试验环境

Kubernetes 是用 minikube 部署的单机版，见 [Minikube 使用手册](../../minikube/index.md)，版本如下：

```sh
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T12:36:28Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:15:22Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```

## ingress-nginx 安装

创建 namespace、configmap，设置 role，部署 deployment（deployment 用到镜像要翻 Q 获取）：

```sh
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.25.1/deploy/static/mandatory.yaml
namespace/ingress-nginx created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
serviceaccount/nginx-ingress-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-role created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
deployment.apps/nginx-ingress-controller created

```

以 nodeport 的方式暴露：

```sh
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.25.1/deploy/static/provider/baremetal/service-nodeport.yaml
```

**如果不能翻Q**，用下面的 yaml：

```sh
$ kubectl apply -f https://raw.githubusercontent.com/introclass/kubernetes-yamls/master/ingress-nginx/ingress-nginx-0.25.1.yaml
```

## 目标服务部署

部署一个目标服务 echo，echo 服务的用途见 [用 echoserver 观察代理/转发效果](../../envoy/echoserver.md)，通过 ingress-nginx 访问该目标服务：

```sh
$ kubectl apply -f https://raw.githubusercontent.com/introclass/kubernetes-yamls/master/all-in-one/echo-all-in-one.yaml
```

在名为 demo-echo 的 namespace 中创建了 pod，并以 nodeport 方式暴露（用来和通过 ingress-nginx 访问的情况对比）：

```sh
$ kubectl -n demo-echo get svc -o wide
NAME   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                     AGE   SELECTOR
echo   NodePort   10.111.29.87   <none>        80:30411/TCP,22:31867/TCP   42d   app=echo

$ kubectl -n demo-echo get pod -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
echo-597d89dcd9-4dp6f   2/2     Running   2          42d   172.17.0.3   minikube   <none>           <none>

$ kubectl -n demo-echo get ingress
NAME                HOSTS                     ADDRESS   PORTS   AGE
ingress-echo        echo.example                        80      42d
```

## 验证环境

通过 nodeport 访问 echo 服务：

```sh
$ curl  http://192.168.99.100:30411

Hostname: echo-597d89dcd9-4dp6f

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008
...
```

通过 nodeport 访问 ingress-nginx：

```sh
$ curl http://192.168.99.100:30933
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>openresty/1.15.8.1</center>
</body>
</html>
```

通过 ingress-nginx 访问 echo 服务：

```sh
$ curl http://192.168.99.100:30933 -H "Host: echo.example"

Hostname: echo-597d89dcd9-4dp6f

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.18
	method=GET
	real path=/
	query=
	request_version=1.1
	request_uri=http://echo.example:8080/
...
```

## 参考

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.github.io/ingress-nginx/examples/ "NGINX Ingress Controller Examples"
[3]: https://github.com/kubernetes/ingress-nginx "ingress-nginx"

<!-- toc -->
# ingress-nginx 的使用方法

[ingress-nginx][3] 是 NGINX Ingress Controller for Kubernetes，它能做的事情越来越多，[NGINX Ingress Controller Examples][2] 列出了 ingress-nginx 能做的事情。

获取后面要使用的 yaml 文件：

```sh
$ git clone https://github.com/introclass/kubernetes-yamls
$ cd kubernetes-yamls/ingress-nginx
$ ls
  01-tls                    05-1-external-svc         08-ratelimit
  02-auth-basic             05-2-auth-oauth-ext       ingress-nginx-0.25.1.yaml
  03-auth-cert              06-rewrite
  04-auth-basic-ext         07-mirror
```

## 试验环境

Kubernetes 是用 minikube 部署的单机版，见 [Minikube 使用手册](../../minikube/index.md)，版本如下：

```sh
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T12:36:28Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:15:22Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```

## 安装 ingress-nginx

创建 namespace、configmap，设置 role，部署 deployment（deployment 用到镜像要翻 Q 获取）：

```sh
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.25.1/deploy/static/mandatory.yaml
```

**如果不能翻Q**，用下面的 yaml：

```sh
$ kubectl apply -f https://raw.githubusercontent.com/introclass/kubernetes-yamls/master/ingress-nginx/ingress-nginx-0.25.1.yaml
```

以 nodeport 的方式暴露 ingress-nginx：

```sh
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.25.1/deploy/static/provider/baremetal/service-nodeport.yaml
```

30933 端口是 http 的映射端口，30358 是 https 协议的映射端口：

```sh
$ kubectl -n ingress-nginx get svc
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.106.78.115   <none>        80:30933/TCP,443:30358/TCP   5d4h
```

## 部署目标服务

部署目标服务 echo，echo 服务的用途见 [用 echoserver 观察代理/转发效果](../../envoy/echoserver.md)，通过 ingress-nginx 访问该目标服务：

```sh
$ kubectl apply -f https://raw.githubusercontent.com/introclass/kubernetes-yamls/master/all-in-one/echo-all-in-one.yaml
```

在名为 demo-echo 的 namespace 中创建了 echo 服务，以 nodeport 方式暴露 echo 服务：

```sh
$ kubectl -n demo-echo get svc -o wide
NAME   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                     AGE   SELECTOR
echo   NodePort   10.111.29.87   <none>        80:30411/TCP,22:31867/TCP   42d   app=echo
```

同时通过 ingress 暴露 echo 服务（用于和 ingress 代理返回的结果对比）：

```sh
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

1. [李佶澳的博客][1]
2. [NGINX Ingress Controller Examples][2]
3. [ingress-nginx][3]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.github.io/ingress-nginx/examples/ "NGINX Ingress Controller Examples"
[3]: https://github.com/kubernetes/ingress-nginx "ingress-nginx"

<!-- toc -->
# Docker 镜像的管理

## 搜索镜像

用 docker search 找镜像： 

* [docker搜索其它registry中的镜像](https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2017/03/22/docker-search-registry.html)

到 Docker Hub 上找镜像：

* [https://hub.docker.com ](https://hub.docker.com/)

	docker search

docker搜索指定registry中的镜像:

	docker search 192.168.1.104:5000/redis
	docker search 192.168.1.104:5000/*

## 镜像的 Tag

docker tag为容器设置tag

	$docker tag 5db5f8471261 ouruser/sinatra:devel

## 下载镜像

	docker pull

## 本地镜像

	[root@localhost ~]# docker images
	REPOSITORY      TAG         IMAGE ID    CREATED     VIRTUAL SIZE
	centos      7.0.1406    3afe3dc5ae15    17 minutes ago  250.1 MB
	<none>      <none>      9d6b25448c7c    3 hours ago     442.9 MB
	ubuntu      14.04       53bf7a53e890    2 days ago      199.8 MB
	ubuntu      latest      53bf7a53e890    2 days ago      199.8 MB

## 镜像文件导入导出

save/load

	docker save IMAGE -o xxx.tar    //将基本镜像一同导出
	docker load -i xxx.tar          //导入

## 制作新镜像

可以通过Dockerfile的形式，使用 docker build 直接建立一个镜像：

[Dockerfile指令](https://docs.docker.com/reference/builder)

也可以把现有的容器提交为镜像:

	[root@localhost ~]# docker commit
	Usage: docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
	Create a new image from a container's changes
	  -a, --author=""     Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
	  -m, --message=""    Commit message
	  -p, --pause=true    Pause container during commit

## 发布镜像

将image发布出去

	docker push 


## 镜像的本地存放

[Docker 镜像管理（一）：本地镜像、本地容器的文件存放目录](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2020/02/08/docker-image-manager.html)

## 制作 base 镜像

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

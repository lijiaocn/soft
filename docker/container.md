<!-- toc -->
# Docker 容器操作

Docker 镜像的管理方法在后面章节，这里先直接用下面的命令获取后面的演示中用到的镜像：

```sh
docker pull alpine:3.9.5
```

[docker command][5] 中列出了所有命令的用法，下面只列举最常用的一些。

## 前台运行

```sh
$ docker run --rm -it  alpine:3.9.5 /bin/sh
/ # ip addr |grep eth0
298: eth0@if299: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
```

docker help run 查看选项：

```sh
--rm  :执行结束后删除容器
-t    :创建tty
-i    :交互运行
```

## 后台运行

```sh
$ docker run -d alpine:3.9.5 /bin/sh -c "while true;do echo hello world; sleep 1;done"
2eecc150eda7af8226b29856468a5428e59664977663779f591cae59c1a217b5
```

-d 表示放入后台运行, 下面显示字符串容器的 id。

## 查看容器

```sh
$ docker ps
CONTAINER ID    IMAGE          COMMAND                  CREATED             STATUS          PORTS    NAMES
d9c9aed4e644    alpine:3.9.5   "/bin/sh -c 'while t…"   57 seconds ago      Up 55 seconds            infallible_antonelli
```

第一栏是容器 ID 的简短形式, 最后一栏 docker 自动为容器分配的名字，可以通过这个名字和容器进行交互。

运行时指定容器名：

```sh
$ docker run -d --name="Hello" alpine:3.9.5 /bin/sh -c "while true;do echo hello world; sleep 1;done"
```

## 容器的输出

用 docker logs 查看后台运行的容器的输出, 目标容器可以通过 id 指定, 也可以通过 name 指定:

```sh
$ docker logs -f Hello
hello world
hello world
hello world
```

## 容器中进程

可以通过 docker top 查看容器内运行的进程：

```sh
$ docker top Hello
PID      USER      TIME     COMMAND
51993    root      0:00     /bin/sh -c while true;do echo hello world; sleep 1;done
52096    root      0:00     sleep 1
```

## 容器详情 

可以通过 docker inspect 查看容器的详情：

```sh
$ docker inspect Hello
[
    {
        "Id": "ef3983bcc0c13660d95d33dce1a4cd7626992a2f211022ac509bae891918918d",
        "Created": "2020-03-01T10:44:06.5211288Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true;do echo hello world; sleep 1;done"
         ...省略...
```

## 进入运行中的容器

docker exec 在指定容器中执行命令，可以通过它进入容器内部：

```sh
$ docker exec -it Hello /bin/sh
/ # ip addr |grep eth0
328: eth0@if329: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
```

## 停止容器 

docker stop停止正在运行的容器：

```sh
$ docker stop Hello 
Hello
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                        PORTS               NAMES
cf591db9d6e9        alpine:3.9.5        "/bin/sh -c 'while t…"   About a minute ago   Exited (137) 55 seconds ago                       Hello
```

容器被停止之后，容器文件依然是存在的。

## 删除容器

删除运行中的容器：

```sh
$ docker rm -f Hello
```

删除已经退出的容器： 

```sh
$ docker rm -f Hello
```

## 端口映射 

在run的-P/-p的选项, 这个选项将镜像的端口映射到宿主机的端口, 这样就可以从外部使用镜像内的服务。

```sh
$ docker run -idt -p 8080:80 nginx:stable
Unable to find image 'nginx:stable' locally
...省略...
```

查看端口映射情况：

```sh
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED             STATUS          PORTS                  NAMES
223c0e90f378   nginx:stable   "nginx -g 'daemon of…"   10 seconds ago      Up 9 seconds    0.0.0.0:8080->80/tcp   friendly_leavitt
```

访问端口：

```sh
$ curl 127.0.0.1:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

## 本地目录挂载

把本地的目录挂载到容器中：

```sh
$ mkdir config
$ echo aaaa >config/a.conf
$ docker run -it --rm -v `pwd`/config:/tmp/ alpine:3.9.5 /bin/sh
```

## 参考

1. [李佶澳的博客][1]
2. [docker的常用操作][2]
3. [docker搜索其它registry中的镜像][3]
4. [从宿主机直接进入docker容器的网络空间][4]
5. [docker command][5]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2017/03/29/docker-usage.html "docker的常用操作"
[3]: https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2017/03/22/docker-search-registry.html "docker搜索其它registry中的镜像"
[4]: https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2017/05/19/docker-enter-net-from-host.html "从宿主机直接进入docker容器的网络空间"
[5]: https://docs.docker.com/engine/reference/run/ "docker command"

<!-- toc -->
# Envoy 安装运行

Envoy 以 docker 镜像的方式分发，社区提供的镜像位于 [envoyproxy][1] 中，常用的有：

* [envoyproxy/envoy-alpine][2] 基于 alpine 的发行镜像
* [envoyproxy/envoy][3] 基于 ubuntu 的发行镜像

获取镜像：

```sh
docker pull envoyproxy/envoy:v1.11.0
```

启动 envoy，用本地的 envoy.yaml 覆盖镜像中的 envoy.yaml：

```sh
docker run -idt --network=host -v `pwd`/envoy.yaml:/etc/envoy/envoy.yaml envoyproxy/envoy:v1.11.0
```

envoy 镜像中包含的命令非常少，可以自己在里面安装一些常用的调试命令，例如：

```sh
apt-get update
apt-get install -y iputils-ping curl iproute
```

本手册使用的是安装了一些常用命令的 envoy 镜像——“lijiaocn/envoy:v1.11.0”，使用下面的 Dockerfile 生成：

```sh
FROM envoyproxy/envoy:v1.11.0
MAINTAINER lijiaocn <lijiaocn@foxmail.com>

RUN apt-get update && apt-get install -y iputils-ping curl iproute net-tools vim
```

用上面的 Docker 制作的镜像比原始的 envoy 镜像体积大，适用于开发调试，镜像已经提交到 docker hub，可以直接拉取使用：

```sh
docker run -idt --network=host -v `pwd`/envoy.yaml:/etc/envoy/envoy.yaml lijiaocn/envoy:v1.11.0
```

## 本手册使用的运行方式

envoy 容器的运行模式根据自己的需要选取，上面使用的是 host 模式，该手册是在 mac 上用非 host 模式运行 envoy 容器，映射了 9901 和 80-86 端口，运行脚本如下：

```sh
IMAGE=lijiaocn/envoy:v1.11.0

if  [ $# -ne 1 ];then
    echo "must choose one config file"
    exit 1
fi

config=$1

if [ `uname`="Darwin" ];then
    docker run -idt -p 9901:9901 -p 80-86:80-86 -v `pwd`/$config:/etc/envoy/envoy.yaml -v `pwd`/log:/var/log/envoy $IMAGE
else
    docker run -idt --network=host -v `pwd`/$config:/etc/envoy/envoy.yaml -v `pwd`/log:/var/log/envoy $IMAGE
fi
```

运行脚本在 [go-code-example/envoydev/xds/envoy-docker-run][4] 中，用 git 获取：

```sh
git clone https://github.com/introclass/go-code-example.git
```

go-code-example 中有多个目录，分别是不同项目的试验素材，本手册使用的代码和脚本位于 envoydev 中

```sh
▾ envoydev/
  ▾ xds/
    ▾ envoy-docker-run/
      ▸ log/
        envoy-0-default.yaml
        envoy-0-example.yaml
        envoy-1-ads-with-xds.yaml
        envoy-1-ads.yaml
        envoy-1-static.yaml
        envoy-1-xds.yaml
        envoy-to-grpc-svc.yaml
        run.sh*
      xds.go
    go.mod
    go.sum
    README.md
```

通过 run.sh 可以很方便地选择一个配置启动，例如： 

```sh
cd  go-code-example/envoydev/xds/envoy-docker-run/
./run.sh envoy-0-example.yaml
```

## 参考

[1]: https://hub.docker.com/u/envoyproxy "docker hub: envoyproxy"
[2]: https://hub.docker.com/r/envoyproxy/envoy-alpine/tags "envoyproxy/envoy-alpine"
[3]: https://hub.docker.com/r/envoyproxy/envoy/tags "envoyproxy/envoy"
[4]: https://github.com/introclass/go-code-example/tree/master/envoydev/xds/envoy-docker-run "envoy-docker-run"

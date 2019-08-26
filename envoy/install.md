# Envoy 安装运行

Envoy 以 docker 镜像的方式分发，社区提供的镜像位于 [envoyproxy][1] 中，常用的有：

* [envoyproxy/envoy-alpine][2]，基于 alpine 的发行镜像
* [envoyproxy/envoy][3] 基于 centos 的发行镜像

获取镜像：

```sh
docker pull envoyproxy/envoy:v1.11.0
```

启动 envoy，用本地的 envoy.yaml 覆盖镜像中的 envoy.yaml：

```sh
docker run -idt --network=host -v `pwd`/envoy.yaml:/etc/envoy/envoy.yaml envoyproxy/envoy:v1.11.0
```

## 本手册使用的运行方式

envoy 的容器的运行模式根据自己的需要选取，上面使用的 host 模式，该手册后续演示使用端口映射的方式，在 mac 上用非 host 模式运行 envoy 容器，映射了 9901 和 80-86 端口：

```sh
IMAGE=envoyproxy/envoy:v1.11.0

if  [ $# -ne 1 ];then
    echo "must choose one config file"
    exit 1
fi

config=$1

if [ `uname`="Darwin" ];then
    docker run -idt -p 9901:9901 -p 84-86:84-86 -v `pwd`/$config:/etc/envoy/envoy.yaml -v `pwd`/log:/var/log/envoy $IMAGE
else
    docker run -idt --network=host -v `pwd`/$config:/etc/envoy/envoy.yaml -v `pwd`/log:/var/log/envoy $IMAGE
fi
```

运行脚本在 [go-code-example/envoydev/xds/envoy-docker-run][4] 中，目录中包含有多个配置文件：

```sh
▾ envoy-docker-run/
  ▸ log/
    envoy-0-default.yaml
    envoy-0-example.yaml
    envoy-1-ads.yaml
    envoy-1-static.yaml
    envoy-1-xds.yaml
    envoy-to-grpc-svc.yaml
    run.sh*
```

通过 run.sh 可以很方便地选择一个配置启动，例如： 

```sh
./run.sh envoy-0-example.yaml
```

## 参考

[1]: https://hub.docker.com/u/envoyproxy "docker hub: envoyproxy"
[2]: https://hub.docker.com/r/envoyproxy/envoy-alpine/tags "envoyproxy/envoy-alpine"
[3]: https://hub.docker.com/r/envoyproxy/envoy/tags "envoyproxy/envoy"
[4]: https://github.com/introclass/go-code-example/tree/master/envoydev/xds/envoy-docker-run "envoy-docker-run"


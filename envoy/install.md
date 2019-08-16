# Envoy 安装运行

Envoy 社区用 docker 镜像的方式分发 envoy，社区提供的镜像位于 [envoyproxy][1] 中，envoy 的分发镜像有：

* [envoyproxy/envoy-alpine][2]，基于 alpine 的发行镜像
* [envoyproxy/envoy][3] 基于 centos 的发行镜像

获取镜像：

```sh
docker pull envoyproxy/envoy:v1.11.0
```

镜像的启动命令为 envoy -c  /etc/envoy/envoy.yaml，因此只需要把镜像中 envoy.yaml 文件替换即可：

```json
"Cmd": [
    "/bin/sh",
    "-c",
    "#(nop) ",
    "CMD [\"envoy\" \"-c\" \"/etc/envoy/envoy.yaml\"]"
],
```

用本地的 envoy.yaml 覆盖镜像中的 envoy.yaml：

```sh
docker run -idt --network=host -v `pwd`/envoy.yaml:/etc/envoy/envoy.yaml envoyproxy/envoy:v1.11.0
```

如果不使用 Host 模式，可以将 envoy 的端口映射到本地。

本手册用的脚本文件和配置文件位于 [go-code-example/envoydev/xds/envoy-docker-run][4]，本手册中使用的 envoy 用到的端口有 9901、80-86，在 mac 上用非 host 模式运行，使用下面的运行方式：

```sh
# ./run.sh <配置文件>
./run.sh envoy-0-default.yaml
```

run.sh 文件内容如下：

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

## 参考

[1]: https://hub.docker.com/u/envoyproxy "docker hub: envoyproxy"
[2]: https://hub.docker.com/r/envoyproxy/envoy-alpine/tags "envoyproxy/envoy-alpine"
[3]: https://hub.docker.com/r/envoyproxy/envoy/tags "envoyproxy/envoy"
[4]: https://github.com/introclass/go-code-example/tree/master/envoydev/xds/envoy-docker-run "envoy-docker-run"

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



[1]: https://hub.docker.com/u/envoyproxy "docker hub: envoyproxy"
[2]: https://hub.docker.com/r/envoyproxy/envoy-alpine/tags "envoyproxy/envoy-alpine"
[3]: https://hub.docker.com/r/envoyproxy/envoy/tags "envoyproxy/envoy"

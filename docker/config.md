<!-- toc -->
# 使用 docker 的建议配置

## 配置阿里云镜像

docker 的镜像服务器在境外，从国内拉取镜像会特别慢。给 docker 配置一个国内的镜像源可以大大缩短镜像的获取时间。

在 /etc/docker/daemon.json 中添加下面的配置，如果该文件不存在则创建， 然后重启 docker：


```json
{
  "registry-mirrors" : [
    "https://pee6w651.mirror.aliyuncs.com"
  ]
}
```

如果 docker 有图形界面更简单了：

![docker daemon 配置](../img/docker/mirror.png)


## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

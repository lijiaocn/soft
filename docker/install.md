<!-- toc -->

# Docker 安装部署

docker、docker-ce 和 docker-ee 的关系见 [moby、docker-ce与docker-ee][1]。

推荐使用 docker-ce。

## CentOS - 安装 docker-ce

添加 docker-ce 的源，安装 docker-ce：

```sh
wget https://download.docker.com/linux/centos/docker-ce.repo
mv docker-ce.repo /etc/yum.repos.d
yum install -y docker-ce
```

启动：

```sh
systemctl start docker
```

## 参考

[1]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2017/07/18/docker-commnuity.html "moby、docker-ce与docker-ee"

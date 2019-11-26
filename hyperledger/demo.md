<!-- toc -->
# HyperLedger Fabric：单机极简部署

这是 Fabric 文档 [Building Your First Network][2] 中提供的做法。Fabric 的示例文件中提供了 一个名为 “byfn.sh” 的脚本 ，这个脚本用 Docker 在本地搭建了一套 Fabric。

用 byfn 搭建的 Fabric 非常不实用，只适合用来学习，但是我发现很多人起步时都采用的这种方式。并且当我们只需要一个简单的 Fabric 环境进行简单验证时，用 byfn 创建比较方便。

实际应用必须采用多节点的方式部署，多节点部署可以参考历史文章：

* [【视频】超级账本HyperLedger：Fabric的全手动、多服务器部署教程](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/04/26/hyperledger-fabric-deploy.html)

## 准备依赖的软件

安装 docker，要掌握用 byfn 搭建 fabric 的方法，必须要对 docker 有基本的了解。

小鸟笔记的 [Docker使用手册](../docker/index.md) 还没开始整理，可以先通过其他材料学习，掌握 Docker 的基本用法即可，譬如：

```sh
docker pull XXXXXX  # 获取 docker 镜像
docker run  XXXX    # 启动容器
docker exec -it XXX # 进入容器
docker rm -f XXX    # 删除容器
```

安装 git，git 是现在最常用的代码管理工具，我们要用 git 从 github 中拉取 fabric-sample 文件。小鸟笔记有一份简陋的 [Git使用手册](../git/index.md)，这里只用 git 获取代码，不需要掌握代码提交、合并等用法，虽然在以后的工作中一定会遇到。

```sh
yum install -y git      # CentOS 系统上
apt-get install -y git  # Ubuntu 系统上
brew install -y git     # Mac 系统上
```

## 下载 Fabric 文件

先创建一个工作目录，文件都存放在这个目录中：

```sh
mkdir ~/hyperledger-fabric-1.4.4
cd ~/hyperledger-fabric-1.4.4
```

下载 fabric 文件。fabric 每个版本的 release 文件都发布在 [fabric releases][3] 上：

![fabric文件下载](../img/fabric/fabric-down.png)

我使用的是 Mac 系统，下载的是 hyperledger-fabric-darwin-amd64-1.4.4.tar.gz：

```sh
$ wget https://github.com/hyperledger/fabric/releases/download/v1.4.4/hyperledger-fabric-darwin-amd64-1.4.4.tar.gz
```

>这个下载过程可能很慢，可能需要翻qiang。

下载 byfn 系列脚本： 

```sh
git clone https://github.com/hyperledger/fabric-samples.git
cd fabric-samples
git checkout -b v1.4.4 # 根据需要切换到对应版本
```

下载解压后得到 bin 和 config 两个目录，bin 目录中是 fabric 的命令文件，config 中 fabric 的配置示例：
```sh
$ tar -xvf hyperledger-fabric-darwin-amd64-1.4.4.tar.gz
$ ls 
bin  config
```

把 bin 目录添加到环境变量 PATH：

```sh
export PATH=$PATH:~/hyperledger-fabric-1.4.4/bin
```

## 创建 first-network

first-network 就是一个最简单 fabric，它的部署文件位于 fabric-samples/first-network：

```sh
cd fabric-samples/first-network
```

进入该目录后，用 byfn.sh 脚本创建：

```sh
./byfn.sh generate
```

如果提示下下面的错误，"cryptogen tool not found. exiting"，那是因为没有把 bin 目录添加到环境变量 PATH 中：

```sh
$ ./byfn.sh generate
Generating certs and genesis block for channel 'mychannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y
proceeding ...
cryptogen tool not found. exiting
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html "Building Your First Network"
[3]: https://github.com/hyperledger/fabric/releases "fabric releases"


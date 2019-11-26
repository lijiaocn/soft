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

## 准备 first-network 需要的证书和文件

first-network 就是一个最简单 fabric，它的部署文件位于 fabric-samples/first-network：

```sh
cd fabric-samples/first-network
```

进入该目录后，用 byfn.sh 创建 first-network 需要的证书文件：

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

执行成功的结果如下，注意 ## 包括的文字，“Generate certificates ...”，这就是为相应组织或者模块生成了证书。

first-network 是由 Org1 和 Org2 两个成员组成的区块链网络，网络中只有一个 Orderer，Org1 有两个 Peer，Org2 有两个 Peer。

```sh
➜  first-network git:(v1.4.4) ./byfn.sh generate
Generating certs and genesis block for channel 'mychannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y
proceeding ...
/Users/lijiao/hyperledger-fabric-1.4.4/bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################
+ cryptogen generate --config=./crypto-config.yaml
org1.example.com
org2.example.com
+ res=0
+ set +x

Generate CCP files for Org1 and Org2
/Users/lijiao/hyperledger-fabric-1.4.4/bin/configtxgen
##########################################################
#########  Generating Orderer Genesis block ##############
##########################################################
CONSENSUS_TYPE=solo
+ '[' solo == solo ']'
+ configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
2019-11-26 22:23:26.404 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-11-26 22:23:26.527 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: solo
2019-11-26 22:23:26.527 CST [common.tools.configtxgen.localconfig] Load -> INFO 003 Loaded configuration: /Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/configtx.yaml
2019-11-26 22:23:26.638 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 004 orderer type: solo
2019-11-26 22:23:26.638 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 005 Loaded configuration: /Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/configtx.yaml
2019-11-26 22:23:26.643 CST [common.tools.configtxgen] doOutputBlock -> INFO 006 Generating genesis block
2019-11-26 22:23:26.644 CST [common.tools.configtxgen] doOutputBlock -> INFO 007 Writing genesis block
+ res=0
+ set +x

#################################################################
### Generating channel configuration transaction 'channel.tx' ###
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
2019-11-26 22:23:26.686 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-11-26 22:23:26.794 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/configtx.yaml
2019-11-26 22:23:26.899 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
2019-11-26 22:23:26.899 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/configtx.yaml
2019-11-26 22:23:26.899 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 005 Generating new channel configtx
2019-11-26 22:23:26.905 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 006 Writing new channel tx
+ res=0
+ set +x

#################################################################
#######    Generating anchor peer update for Org1MSP   ##########
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
2019-11-26 22:23:26.943 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-11-26 22:23:27.049 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/configtx.yaml
2019-11-26 22:23:27.165 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
2019-11-26 22:23:27.165 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/configtx.yaml
2019-11-26 22:23:27.165 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005 Generating anchor peer update
2019-11-26 22:23:27.166 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006 Writing anchor peer update
+ res=0
+ set +x

#################################################################
#######    Generating anchor peer update for Org2MSP   ##########
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP
2019-11-26 22:23:27.206 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-11-26 22:23:27.347 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/configtx.yaml
2019-11-26 22:23:27.473 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
2019-11-26 22:23:27.473 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/configtx.yaml
2019-11-26 22:23:27.473 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005 Generating anchor peer update
2019-11-26 22:23:27.474 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006 Writing anchor peer update
+ res=0
+ set +x
```

## 启动 first-network

启动过程拉取 fabric 的 docker 镜像，在国内镜像的拉取可能特别慢，建议先给 docker [配置镜像源](../docker/config.md) ：

first-network 启动命令：

```sh
$ ./byfn.sh up
Starting for channel 'mychannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y
proceeding ...
Unable to find image 'hyperledger/fabric-tools:latest' locally
latest: Pulling from hyperledger/fabric-tools
7ddbc47eeb70: Pulling fs layer
c1bbdc448b72: Pulling fs layer

...省略...

90
===================== Query successful on peer1.org2 on channel 'mychannel' =====================

========= All GOOD, BYFN execution completed ===========


 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

出现 All GOOD 后，启动成功。



## first-network 的组成

用 docker ps 可以正在运行的容器，最后一列是容器的名称：

```sh
➜  first-network git:(v1.4.4) ✗ docker ps
CONTAINER ID        IMAGE                                            COMMAND                  CREATED             STATUS              PORTS                      NAMES
b2d7c1e32b74        dev-peer1.org2.example.com-mycc-1.0-26c2ef3283   "chaincode -peer.add…"   5 minutes ago       Up 5 minutes                                   dev-peer1.org2.example.com-mycc-1.0
4e3b06b72953        dev-peer0.org1.example.com-mycc-1.0-384f11f484   "chaincode -peer.add…"   5 minutes ago       Up 5 minutes                                   dev-peer0.org1.example.com-mycc-1.0
6aebf7da1213        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce   "chaincode -peer.add…"   6 minutes ago       Up 6 minutes                                   dev-peer0.org2.example.com-mycc-1.0
bb5b01488fe5        hyperledger/fabric-tools:latest                  "/bin/bash"              7 minutes ago       Up 7 minutes                                   cli
7781e5181954        hyperledger/fabric-peer:latest                   "peer node start"        7 minutes ago       Up 7 minutes        0.0.0.0:7051->7051/tcp     peer0.org1.example.com
d5ae822f095a        hyperledger/fabric-orderer:latest                "orderer"                7 minutes ago       Up 7 minutes        0.0.0.0:7050->7050/tcp     orderer.example.com
844929c28283        hyperledger/fabric-peer:latest                   "peer node start"        7 minutes ago       Up 7 minutes        0.0.0.0:10051->10051/tcp   peer1.org2.example.com
5ff858ab54cc        hyperledger/fabric-peer:latest                   "peer node start"        7 minutes ago       Up 7 minutes        0.0.0.0:9051->9051/tcp     peer0.org2.example.com
7ab6831f4530        hyperledger/fabric-peer:latest                   "peer node start"        7 minutes ago       Up 7 minutes        0.0.0.0:8051->8051/tcp     peer1.org1.example.com
```

组成 first-network 的容器：

```sh
orderer.example.com        生成保证区块顺序的 orderer 
peer0.org1.example.com     org1 的第一个 peer
peer1.org1.example.com     org1 的第二个 peer
peer0.org2.example.com     org2 的第一个 peer
peer1.org2.example.com     org2 的第二个 peer
```


下面三个容器是 byfn.sh 脚本创建的一个名为 mycc 的智能合约，fabric 的智能合约是用 docker 运行的：

```sh
dev-peer1.org2.example.com-mycc-1.0
dev-peer0.org1.example.com-mycc-1.0
dev-peer0.org2.example.com-mycc-1.0
```


cli 使用用来查看、管理 first-network 的容器，用下面的方式进入：

```sh
$ docker exec -it cli /bin/bash
root@bb5b01488fe5:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

在容器内执行 ls，可以看到创建 first-network 时用到文件：

```sh
$ ls -lh
total 28K
drwxr-xr-x  7 root root 224 Nov 26 14:23 channel-artifacts
drwxr-xr-x  4 root root 128 Nov 26 14:23 crypto
-rw-r--r--  1 root root   3 Nov 26 14:34 log.txt
-rw-r--r--  1 root root 21K Nov 26 14:31 mychannel.block
drwxr-xr-x 10 root root 320 Nov 25 14:52 scripts
```

## first-network 是怎么回事？

这个问题三言两语还真是说不清楚。别看创建 first-network 特别简单，但是 byfn.sh 背后的工作有很多！用 docker-compose + 一堆脚本。

强烈建议通过下面教程学习 fabric 的组成。这个教程使用的是 fabric 1.1 （两年前的版本），但基本的原理不变，把每个组件的用途和配置方法讲清楚：

* [【视频】超级账本HyperLedger：Fabric的全手动、多服务器部署教程](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/04/26/hyperledger-fabric-deploy.html)

(最近忙，没时间制作最新的教程，2019-11-26 22:58:10)

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html "Building Your First Network"
[3]: https://github.com/hyperledger/fabric/releases "fabric releases"


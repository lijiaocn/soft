<!-- toc -->
# HyperLedger Fabric 的后续安排

接下来要详细说明 Fabric 每一个组件的用法，Fabric 系统的用法，如何创建、执行合约之类的，以及 HyperLedger 旗下其它项目的使用，譬如 Cello......

工作量有点大，近期的主要精力要放到工作上正在用的 [Kubernetes](../k8s/index.md) 和准备用的 [Istio](../istio/index.md)，HyperLedger 的后续跟进比较窘迫......

检视了一下之前的教程和笔记，核心逻辑还是对，虽然使用的是 Fabric 1.1 和 Fabric 1.2，但不妨碍对 Fabric 组件和运作方式的理解。笔记已经很详细，网易云课堂上的视频可用可不用。
掌握一个系统，最好的方法就是自己折腾，下面这些笔记和视频都是辅助，我会尽力把最新的内容更新到这本「小鸟笔记」中（2019-11-27 08:57:51)。


**视频演示：**

* [【视频】Fabric的全手动、多服务器部署教程](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/04/26/hyperledger-fabric-deploy.html)
* [【视频】使用Ansible进行Fabric多节点分布式部署（实战）](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/07/09/hyperledger-fabric-ansible-deploy.html)
* [【视频】Fabric从1.1.0升级到1.2.0](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/07/19/hyperledger-fabric-1-2-0.html)
* [【视频】Fabric使用kafka进行区块排序（共识）](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/07/28/hyperledger-fabric-orderer-kafka.html)
* [【视频】为Fabric的Peer节点配置CouchDB](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/07/19/hyperledger-fabric-with-couchdb.html)
* [【视频】Fabric-CA的使用演示(两个组织一个Orderer三个Peer)](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/05/04/fabric-ca-example.html)
* [【视频】Fabric的Chaincode（智能合约、链码）开发、使用演示](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/07/17/hyperledger-fabric-chaincodes-example.html)
* [【视频】Fabric Go SDK的使用](https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/07/28/hyperledger-fabric-sdk-go.html)
* [【视频】Fabric nodejs SDK的使用](https://www.lijiaocn.com/%E7%BC%96%E7%A8%8B/2018/04/25/hyperledger-fabric-sdk-nodejs.html)
* [【视频】Fabric进阶，在已有的Channel中添加新的组织](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/06/18/hyperledger-fabric-add-new-org.html)
* [【视频】超级账本HyperLedger：Fabric源码走读(零)：源代码阅读环境准备](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/07/17/hyperledger-fabric-source-code.html)

**文字介绍：**

* [超级账本工作组旗下项目介绍](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/05/08/hyperledger-projects-intro.html)
* [Fabric掰开揉碎，一文解惑](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/06/25/hyperledger-fabric-main-point.html)
* [Fabric的基本概念与基础用法](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/02/23/hyperledger-fabric-usage.html)
* [FabricCA的基本概念与用法讲解](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/04/27/hyperledger-fabric-ca-usage.html)
* [FabricCA的级联使用（InterMediateCA）](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/07/09/hyperledger-fabric-ca-cascade.html)
* [Fabric Chaincode（智能合约、链码）开发方法](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/05/05/hyperledger-fabric-chaincode.html)
* [Fabric Channel配置的读取转换](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/06/19/hyperledger-channel-config-operation.html)
* [Explorer安装使用](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/04/26/hyperledger-explorer.html)
* [Cello部署和使用](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/04/25/hyperledger-cello.html)

**问题汇总：**

* [Fabric部署过程时遇到的问题汇总](https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2018/04/25/hyperledger-fabric-problem.html)
* [Fabric的Chaincode开发过程中遇到的问题](https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2018/07/20/hyperledger-fabric-chaincode-problem.html)
* [Fabric Node.js SDK使用时遇到的问题](https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2018/07/15/hyperledger-fabric-nodejs-problem.html)
* [Fabric Golang SDK使用时遇到的问题](https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2018/07/15/hyperledger-fabric-golang-problem.html)
* [Fabric 1.2.0使用时遇到的问题](https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2018/07/25/hyperledger-fabric-1-2-0-problems.html)

更多文章：

* [超级账本&区块链实践文章（持续更新）](https://www.lijiaocn.com/tags/blockchain.html)

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"

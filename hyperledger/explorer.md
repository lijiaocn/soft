<!-- toc -->
# HyperLedger Fabric 区块链浏览器的部署方法

[blockchain-explorer][2] 是一个独立项目，可以用来查看 fabric 内的成员和区块。

## 安装依赖的软件

explorer 依赖 nodejs 8.11（高版本的不支持）、PostgreSQL 9.5和以上版本、jq 命令。

下面的操作是在 mac 上进行的，除了依赖软件的安装方式，大部分操作和在 linux 上操作相同：

[超级账本HyperLedger：Explorer安装使用][3]

### PostgreSQL 安装设置

如果对 PostgreSQL 完全不了解，可以先学习：

* [PostgresSQL数据库的基本使用——新手入门][4]

这里为了屏蔽不同操作系统上的差异，用 docker 启动 PostgreSQL 数据库，这个过程拉取 Docker 镜像，[单机极简部署](./demo.md) 中提到过 docker 的使用方法和拉取镜像慢的问题：

```sh
$ docker run -idt \
    -e POSTGRES_USER="postgres" \
    -e POSTGRES_PASSWORD="password" \
    -p 5432:5432  \
    postgres:latest
```

本地还需要安装 psql，一个用于连接 postgres 数据库的命令：

```sh
$ brew cask install postgresql               # Mac 上安装方法
$ yum install -y epel-release  postgresql    # CentOS 上安装方法
$ apt-get install  postgresql                # Ubuntu 上安装方法
```

确定能用下面的命令进入数据库，密码是 password：

```sh
$ psql -h 127.0.0.1 -U postgres
Password for user postgres:
psql (11.2, server 12.1 (Debian 12.1-1.pgdg100+1))
WARNING: psql major version 11, server major version 12.
         Some psql features might not work.
Type "help" for help.

postgres=#
```

## 安装 nodejs 

exploere 要求 nodejs 的版本不能太高，必须是 8.x 版本。

在 CentOS 上直接下载安装：

```sh
yum erase nodejs
wget https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-x64.tar.xz
tar -xvf node-v8.11.3-linux-x64.tar.xz
cd node-v8.11.3-linux-x64
cp -rf * /usr/
```

在 Mac 上用下面的方法安装 node 8.11.3：


```sh
wget https://nodejs.org/dist/v8.11.3/node-v8.11.3.pkg
open node-v8.11.3.pkg
```

确保 node 的版本是 8.11.3：

```sh
$ node --version
v8.11.3
```

>特别注意：node 版本过高或过低，都会出现各种奇葩问题！

## 下载 blockchain-explorer 文件

把 blockchain-explorer 下载到 ~/hyperledger-fabric-1.4.4 目录中：

```sh
$ cd ~/hyperledger-fabric-1.4.4
$ git clone https://github.com/hyperledger/blockchain-explorer.git
```

这里使用的是 v0.3.9.5 版本，explorer 和 fabric 的版本适配情况见 [blockchain-explorer release][5]：

```sh
$ cd blockchain-explorer
$ git checkout -b v0.3.9.5    # v0.3.9.5 适配 fabric 1.4
```

## 核实配置文件 app/explorerconfig.json

核实配置文件 app/explorerconfig.json，确定里面的数据库用户名和密码是正确的：

```json
{
  "persistence": "postgreSQL",
  "platforms": ["fabric"],
  "postgreSQL": {
    "host": "127.0.0.1",
    "port": "5432",
    "database": "fabricexplorer",
    "username": "hppoc",
    "passwd": "password"
  },
  "sync": {
    "type": "local",
    "platform": "fabric",
    "blocksSyncTime": "3"
  }
}
```

## 初始化 blockchain-explorer 使用的数据库 

创建数据库：

```sh
$ cd app/persistence/fabric/postgreSQL/db
$ ./createdb.sh
```

我在 Mac 上运行时遇到下面错误：

```sh
$ ./createdb.sh
Copying ENV variables into temp file...
USER="hppoc"
DATABASE="fabricexplorer"
PASSWD='password'
Executing SQL scripts...
psql: could not connect to server: No such file or directory
	Is the server running locally and accepting
	connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
psql: could not connect to server: No such file or directory
	Is the server running locally and accepting
	connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

这是因为我的 postgre 是用 docker 启动的，访问时需要指定 IP 地址和用户，需要修改 createdb.sh。

在 Mac 上：

```sh
darwin*) psql postgres -v dbname=$DATABASE -v user=$USER -v passwd=$PASSWD -f ./explorerpg.sql ;
psql postgres -v dbname=$DATABASE -v user=$USER -v passwd=$PASSWD -f ./updatepg.sql ;;
```

修改为：

```sh
darwin*) psql postgres -h 127.0.0.1 -U postgres -v dbname=$DATABASE -v user=$USER -v passwd=$PASSWD -f ./explorerpg.sql ;
psql postgres -h 127.0.0.1 -U postgres -v dbname=$DATABASE -v user=$USER -v passwd=$PASSWD -f ./updatepg.sql ;;
```

在 Linux 上修改方法类似，就是加上 -h 127.0.0.1 -U postgres，我没试验，使用 linux 系统的同学可以自己试一下：

```sh
linux*) psql postgres -h 127.0.0.1 -U postgres -v dbname=$DATABASE -v user=$USER -v passwd=$PASSWD -f ./explorerpg.sql ;
psql postgres -h 127.0.0.1 -U postgres -v dbname=$DATABASE -v user=$USER -v passwd=$PASSWD -f ./updatepg.sql ;;
```

修改后重新执行 ./createdb.sh， 这时候会提示输入的密码，密码就是前面用 docker 启动数据库时设置的 password。
输出的最后一行为下面的内容时，数据设置成功：

```sh
You are now connected to database "fabricexplorer" as user "postgres".
```

## 设置 fabric 对接文件

blockchain-explorer 中的 app/platform/fabric/config.json 是用来连接 fabric 的配置文件：

```sh
$ ls app/platform/fabric/config.json
app/platform/fabric/config.json
```

查看 config.json 文件的内容，你会发现这个文件中的配置针对正好是 [上一节](./demo.md) 中创建的 first-network：

```sh
$ cat app/platform/fabric/config.json |grep peer
          "peers": {
            "peer0.org1.example.com": {}
              "peer": {
            "path": "/fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore"
            "path": "/fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts"
            "path": "/fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/keystore"
      "peers": {
        "peer0.org1.example.com": {
            "path": "/fabric-path/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
            "ssl-target-name-override": "peer0.org1.example.com"
        "peer1.org1.example.com": {
        "peer0.org2.example.com": {
        "peer1.org2.example.com": {
```

但是 config.json 中的文件路径为 “/fabric-path/fabric-samples...."，和这里的文件路径不同，需要全部修改为：

```sh
"path": "/Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore"
```

注意必须使用绝对路径，路径不能是 `~`，我这里是 /Users/lijiao，记得换成你自己的路径，用下面的命令一次修改完成，：

```sh
# Mac 上执行：
$ sed -i "" -e "s#/fabric-path#/Users/lijiao/hyperledger-fabric-1.4.4#" app/platform/fabric/config.json
# Linux 上执行：
$ sed -i -e "s#/fabric-path#/Users/lijiao/hyperledger-fabric-1.4.4#" app/platform/fabric/config.json
```

## 编译启动 blockchain-explorer

安装依赖包，构建 blockchain-explorer：

```sh
$ npm config set registry https://registry.npm.taobao.org  # 添加淘宝镜像源头，加快速度
$ cd blockchain-explorer
$ npm install
$ cd client   # 进入 client 目录
$ npm install
$ npm run build
```

构建成功输出：

![explorer-client-build 成功](../img/fabric/explorer-client-build.png)

启动：

```
$ cd blockchain-explorer
$ node ./main.js
....显示信息...
write_set size >>>>>>>>> :  0.5014553070068359 MB
Insert sql is INSERT INTO transactions  ( "blockid","txhash","createdt","chaincodename","chaincode_id","status","creator_msp_id","endorser_msp_id","type","read_set","write_set","channel_genesis_hash","validation_code","envelope_signature","payload_extension","creator_nonce","chaincode_proposal_input","endorser_signature","creator_id_bytes","payload_proposal_hash","endorser_id_bytes" ) VALUES( $1,$2,$3,$4,$5,$6,$7,$8,$9,$10,$11,$12,$13,$14,$15,$16,$17,$18,$19,$20,$21  ) RETURNING *;
... 省略 ...
```

在浏览器中打开 127.0.0.1:8080，监听地址在 appconfig.json 中配置：

![Fabric 区块链浏览器页面](../img/fabric/explorer.png)


## 遇到的一些错误

### 数据库密码不对

如果配置文件 app/explorerconfig.json 中数据库账号密码不对，node ./main.js 会报下面的错误：

```sh
postgres://hppoc:pass12345@127.0.0.1:5432/fabricexplorer
error when connecting to db: { error: password authentication failed for user "hppoc"
    at Connection.parseE (/Users/lijiao/hyperledger-fabric-1.4.4/blockchain-explorer/node_modules/pg/lib/connection.js:554:11)
    at Connection.parseMessage (/Users/lijiao/hyperledger-fabric-1.4.4/blockchain-explorer/node_modules/pg/lib/connection.js:379:19)
    at Socket.<anonymous> (/Users/lijiao/hyperledger-fabric-1.4.4/blockchain-explorer/node_modules/pg/lib/connection.js:119:22)
    at emitOne (events.js:116:13)
    at Socket.emit (events.js:211:7)
    at addChunk (_stream_readable.js:263:12)
    at readableAddChunk (_stream_readable.js:250:11)
    at Socket.Readable.push (_stream_readable.js:208:10)
    at TCP.onread (net.js:601:20)
```

注意 exploer 优先从环境变量中获取数据库的账号密码，如果文件中的账号密码是正确的，但是环境变量中的不正确，也会报错，用下面命令清空环境变量：

```sh
unset DATABASE_HOST
unset DATABASE_PORT
unset DATABASE_DATABASE
unset DATABASE_USERNAME
unset DATABASE_PASSWD
```

### Faric 的对接文件配置错误

node ./main.js  打印下面的字符，然后自动退出：

```sh
**************************************************************************************
Error : Failed to connect client peer, please check the configuration and peer status
Info :  Explorer will continue working with only DB data
**************************************************************************************
```

如果 app/platform/fabric/config.json 中的 path 路径配置错了，会出现这种问题，注意必须使用完整的绝对路径，不能用 `~/xxxxx`：

```sh
$ cat  app/platform/fabric/config.json |grep path
            "path": "./tmp/credentialStore_Org1/credential",
              "path": "./tmp/credentialStore_Org1/crypto"
          "fullpath": false,
            "path": "/Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore"
            "path": "/Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts"
            "path": "/Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/keystore"
            "path": "/Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/keystore"
            "path": "/Users/lijiao/hyperledger-fabric-1.4.4/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://github.com/hyperledger/blockchain-explorer "blockchain-explorer"
[3]: https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/04/26/hyperledger-explorer.html "超级账本HyperLedger：Explorer安装使用"
[4]: https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2017/08/31/postgre-usage.html "PostgresSQL数据库的基本使用——新手入门"
[5]: https://github.com/hyperledger/blockchain-explorer/releases "blockchain-explorer release"

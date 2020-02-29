<!-- toc -->
# MySQL 的常规使用

MySQL 被 Oracle 收购后，分叉成了隶属 Oracle 的 MySQL 分支，和社区维护的 MariaDB 分支。在 CentOS 等开源系统上，现在通常默认提供的是 MariaDB。

## MariaDB Docker 镜像的使用 

Docker 社区维护了一个 mariadb 镜像：

```sh
$ docker search mariadb
NAME                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mariadb                                MariaDB is a community-developed fork of MyS…   3263                [OK]
```

镜像的使用方法见 [mariadb image usage][3]，镜像构建文件见 [docker-library/mariadb][2]。 

下面以 mariadb:10.4.1 为例：

```sh
$ docker push mariadb:10.4.1
```

### 镜像的环境变量

```sh
MYSQL_ROOT_PASSWORD            # root 密码
MYSQL_DATABASE                 # 启动时创建一个 database
MYSQL_USER, MYSQL_PASSWORD     # 启动时增加一个特权用户
MYSQL_ALLOW_EMPTY_PASSWORD     # 允许空密码 
MYSQL_RANDOM_ROOT_PASSWORD     # 随机生成 root 密码
```

### 镜像的关键目录

**/docker-entrypoint-initdb.d** ：镜像启动时会根据上面的环境变量，完成数据库、用户和密码后，然后执行该目录中 `.sh`、`.sql`、`.sql.gz` 文件，可以把初始化脚步挂载到这个目录中，按照文件名称排序执行。

>特别注意，只有第一次启动，/var/lib/mysql 目录为空的时候才会加载执行。

**/var/lib/mysql**：数据库文件的目录。

**/etc/mysql/my.cnf**：数据库的配置文件。

**/etc/mysql/conf.d**：默认的 my.cnf 文件的配置加载目录。

### 镜像的命令行参数

```sh
$ docker run -it --rm mariadb:10.4.1 --verbose --help
```

### 镜像的使用示范

Docker Compose 启动：

```yaml
version: "3"
services:
  mysql:
    image: mariadb:10.4.1
    environment:
      - TZ=CST-8
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=testdb
      - MYSQL_USER=testdb
      - MYSQL_PASSWORD=testdb
    volumes:
      #本地文件目录
      - ./data/mysql:/var/lib/mysql
      - ./conf/sql:/docker-entrypoint-initdb.d
    ports:
      - "13306:3306"
```

执行 sql:

```sh
$ mysql -u testdb -ptestdb -h 127.0.0.1 -P 3306 -D testdb< ./sql/insert.sql
```

或者：

```sh
$ docker exec -i mysql sh -c 'exec mysql -utestdb -ptestdb -D testdb' < ./sql/insert.sql
```

导出数据（只导出表结构 -d，只导表数据 -t）：

```sh
$ docker exec mysql sh -c 'exec mysqldump -utestdb -ptestdb testdb'  >backup.sql
```

数据重新加载方法和执行 sql 相同。


## 参考

1. [李佶澳的博客][1]
2. [docker-library/mariadb][2]
3. [MariaDB Image Usage][3]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://github.com/docker-library/mariadb "docker-library/mariadb"
[3]: https://hub.docker.com/_/mariadb/ "MariaDB Image Usage"

<!-- toc -->
# 常用软件在 Mac 上的安装和使用

## 数据库图形工具

mysql 图形界面管理工具：sequel-pro、mysqlworkbench：

```sh
brew cask install sequel-pro mysqlworkbench
```

通用的数据库图形界面工具：navicat-premium

```sh
brew cask install navicat-premium
```

[Getting Started with PostgreSQL on Mac OSX][4] 列出了几个 postgres 的图形界面管理工具:

* Postico
* pgAdmin

## postgres

[PostgreSQL][2] 好像越来越流行了。

推荐：

* [Getting Started with PostgreSQL on Mac OSX][4]

以前的笔记：

* [PostgresSQL数据库新手入门](https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2017/08/31/postgre-usage.html)
* [新建用户怎样才能用密码登陆？](https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2018/09/28/postgres-user-manage.html)

### 安装 postgres

在 Mac 上用 brew 安装：

```sh
$ brew search postgres
postgresql@11   postgresql@10   postgresql@9.4    postgresql@9.5   postgresql@9.6

$ brew install postgresql@11
```

安装完成后显示操作提示：

```sh
To migrate existing data from a previous major version of PostgreSQL run:
  brew postgresql-upgrade-database

postgresql@11 is keg-only, which means it was not symlinked into /usr/local,
because this is an alternate version of another formula.

If you need to have postgresql@11 first in your PATH run:
  echo 'export PATH="/usr/local/opt/postgresql@11/bin:$PATH"' >> ~/.zshrc

For compilers to find postgresql@11 you may need to set:
  export LDFLAGS="-L/usr/local/opt/postgresql@11/lib"
  export CPPFLAGS="-I/usr/local/opt/postgresql@11/include"

For pkg-config to find postgresql@11 you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/postgresql@11/lib/pkgconfig"


To have launchd start postgresql@11 now and restart at login:
  brew services start postgresql@11
Or, if you don't want/need a background service you can just run:
  pg_ctl -D /usr/local/var/postgresql@11 start
```

设置环境变量：

```sh
echo 'export PATH="/usr/local/opt/postgresql@11/bin:$PATH"' >> ~/.zshrc
```

验证版本：

```sh
$ postgres -V
postgres (PostgreSQL) 11.6
```

### 命令行工具的单独安装

如果只是要从本地访问 postgres，可以只安装命令行工具：

```sh
$ brew install pgcli
...
If you need to have libpq first in your PATH run:
  echo 'export PATH="/usr/local/opt/libpq/bin:$PATH"' >> ~/.zshrc

For compilers to find libpq you may need to set:
  export LDFLAGS="-L/usr/local/opt/libpq/lib"
  export CPPFLAGS="-I/usr/local/opt/libpq/include"

For pkg-config to find libpq you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/libpq/lib/pkgconfig"
```

### 启动 postgres

启动 postgres：

```sh
$ brew services start postgresql@11
==> Successfully started `postgresql@11` (label: homebrew.mxcl.postgresql@11)
```

查看状态：

```sh
$ brew services list |grep postgres
postgresql@11 started lijiao /Users/lijiao/Library/LaunchAgents/homebrew.mxcl.postgresql@11.plist
```

默认数据库文件路径：

```sh
$ ls /usr/local/var/postgresql@11
PG_VERSION           pg_ident.conf        pg_snapshots         pg_wal
base                 pg_logical           pg_stat              pg_xact
global               pg_multixact         pg_stat_tmp          postgresql.auto.conf
pg_commit_ts         pg_notify            pg_subtrans          postgresql.conf
pg_dynshmem          pg_replslot          pg_tblspc            postmaster.opts
pg_hba.conf          pg_serial            pg_twophase          postmaster.pid
```

### 第一次登陆

本地登陆 postgres：

```sh
$ psql postgres
psql (11.6)
Type "help" for help.

postgres=#
```

默认创建的 role（用户）：

```sh
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 lijiao    | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

所在的系统的当前用户会被自动创建为 postgres 的超级用户，所以在本地可以直接用 `psql postgres` 登陆。

### 创建其它用户

创建一个新用户：

```sql
create user postgresdemo with password 'password123';
```

在本地用新用户登陆（注意指定 -h 127.0.0.1 -p 5432）：

```sh
$ psql -h 127.0.0.1 -p 5432 -U postgresdemo
Password:
psql (11.6)
Type "help" for help.

postgres=>
```

本地登陆时，可能无需密码就成功了，远程登陆时可能密码正确也无法登陆，这是 postgres 的认证配置导致的：

```sh
$ cat /usr/local/var/postgresql@11/pg_hba.conf |grep all
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
local   replication     all                                     trust
host    replication     all             127.0.0.1/32            trust
host    replication     all             ::1/128                 trust
```

默认对本地全部信任（`trust`），没有配置其它来源访问。

用下面的配置允许 postgresdemo 用户从任何地址访问所有数据库，通过密码认证：

```sh
# TYPE  DATABASE   USER          ADDRESS      METHOD
  host  all        postgresdemo  0.0.0.0/0    password
```

添加配置后需要重启 postgresql，详细说明见：[ Postgres 新建用户怎样才能用密码登陆？](https://www.lijiaocn.com/%E6%8A%80%E5%B7%A7/2018/09/28/postgres-user-manage.html)

### 创建数据库

创建数据库并授权给 postgresdemo：

```mysql
create database postgresdemo;
grant all on database  postgresdemo to postgresdemo;
```

如果要限制该数据库的访问方式，可以在 pg_hba.conf 添加类似配置：

```sh
# TYPE  DATABASE        USER            ADDRESS      METHOD
  host  postgresdemo    postgresdemo    0.0.0.0/0    password
```

数据库操作:

```sh
\list: lists all the databases in Postgres
\connect: connect to a specific database
\dt: list the tables in the currently connected database
```

## 参考

1. [李佶澳的博客][1]
2. [PostgreSQL][2]
3. [PostgreSQL 使用方法][3]
4. [Getting Started with PostgreSQL on Mac OSX][4]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.postgresql.org/ "PostgreSQL"
[3]: https://www.lijiaocn.com/tags/all.html#postgres "postgres usage"
[4]: https://www.codementor.io/@engineerapart/getting-started-with-postgresql-on-mac-osx-are8jcopb "Getting Started with PostgreSQL on Mac OSX"

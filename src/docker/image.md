<!-- toc -->
# Docker 镜像的管理

## 搜索镜像

docker search 默认搜索 docker hub 中的镜像：

```sh
$ docker search nginx
```

如果搜索其它 registry 中的镜像，可用下面的 [方法][2]:

```sh
docker search 192.168.1.104:5000/redis
docker search 192.168.1.104:5000/*
```

到 Docker Hub 上找镜像、查看镜像详情：[https://hub.docker.com ](https://hub.docker.com/)。

## 镜像的 tag

每个镜像都有 tag，tag 是镜像的版本，我们可以根据自己需要为镜像重新命名，生成新的 tag：

```sh
docker tag alpine:3.9.5 myalpine:1.0
```

## 下载镜像

```sh
$ docker pull alpine:3.9.5
```

## 查看镜像

```sh
$ docker images
REPOSITORY      TAG     IMAGE ID        CREATED       VIRTUAL SIZE
alpine         3.9.5    82f67be598eb    5 weeks ago   5.53MB
myalpine       1.0      82f67be598eb    5 weeks ago   5.53MB
```

查看镜像详情：

```sh
$ docker inspect  alpine:3.9.5
```

## 镜像的导入导出

用 save 命令把镜像导出到文件中：

```sh
$ docker save alpine:3.9.5 -o alpine.tar

```

用 load 命令加载： 

```sh
$ docker load -i alpine.tar
```

## 制作新镜像

方法一：编写 Dockerfile，使用 docker build 直接建立一个镜像。

* [Dockerfile 语法](https://docs.docker.com/reference/builder)

[docker-containers](https://github.com/introclass/docker-containers.git) 中提供不少实用镜像 Dockerfile，可以参考：

```sh
git clone https://github.com/introclass/docker-containers.git
```

方法二：把现有的容器做成镜像。

```sh
$ docker commit
Usage: docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
Create a new image from a containers changes
  -a, --author=""     Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -m, --message=""    Commit message
  -p, --pause=true    Pause container during commit
```

例如：

```sh
$ docker commit Hello myhello:latest
sha256:72c60d1495909cada678ee468171828c8fccf4b41712a96bc43017139e35568e
```

方法三：制作 BASE 镜像。

前面两种做法都需要先有一个镜像，在已有镜像的基础上制作新镜像，那么怎样凭空创建一个镜像呢？

[Create a base image](https://docs.docker.com/develop/develop-images/baseimages/) 介绍了两种方法：

```sh
#!/usr/bin/env bash
#
# Create a base CentOS Docker image.
#
# This script is useful on systems with yum installed (e.g., building
# a CentOS image on CentOS).  See contrib/mkimage-rinse.sh for a way
# to build CentOS images on other systems.

usage() {
    cat <<EOOPTS
$(basename $0) [OPTIONS] <name>
OPTIONS:
  -y <yumconf>  The path to the yum config to install packages from. The
                default is /etc/yum.conf.
EOOPTS
    exit 1
}

# option defaults
yum_config=/etc/yum.conf
while getopts ":y:h" opt; do
    case $opt in
        y)
            yum_config=$OPTARG
            ;;
        h)
            usage
            ;;
        \?)
            echo "Invalid option: -$OPTARG"
            usage
            ;;
    esac
done
shift $((OPTIND - 1))
name=$1

if [[ -z $name ]]; then
    usage
fi

#--------------------

target=$(mktemp -d --tmpdir $(basename $0).XXXXXX)

set -x

# 创建系统的必须的设备文件
mkdir -m 755 "$target"/dev 
mknod -m 600 "$target"/dev/console c 5 1
mknod -m 600 "$target"/dev/initctl p
mknod -m 666 "$target"/dev/full c 1 7
mknod -m 666 "$target"/dev/null c 1 3
mknod -m 666 "$target"/dev/ptmx c 5 2
mknod -m 666 "$target"/dev/random c 1 8
mknod -m 666 "$target"/dev/tty c 5 0
mknod -m 666 "$target"/dev/tty0 c 4 0
mknod -m 666 "$target"/dev/urandom c 1 9
mknod -m 666 "$target"/dev/zero c 1 5

# 安装root文件系统
yum -c "$yum_config" --installroot="$target" --releasever=/ --setopt=tsflags=nodocs \
    --setopt=group_package_types=mandatory -y groupinstall Core
yum -c "$yum_config" --installroot="$target" -y clean all

# 网络配置
cat > "$target"/etc/sysconfig/network <<EOF
NETWORKING=yes
HOSTNAME=localhost.localdomain
EOF

# 删除不必要的文件
# effectively: febootstrap-minimize --keep-zoneinfo --keep-rpmdb
# --keep-services "$target".  Stolen from mkimage-rinse.sh
#  locales
rm -rf "$target"/usr/{lib,share}/locale,{lib,lib64}/gconv,bin/localedef,sbin/build-locale-archive}
#  docs
rm -rf "$target"/usr/share/{man,doc,info,gnome/help}
#  cracklib
rm -rf "$target"/usr/share/cracklib
#  i18n
rm -rf "$target"/usr/share/i18n
#  sln
rm -rf "$target"/sbin/sln
#  ldconfig
rm -rf "$target"/etc/ld.so.cache
rm -rf "$target"/var/cache/ldconfig/*

version=
if [ -r "$target"/etc/redhat-release ]; then
    version="$(sed 's/^[^0-9\]*\([0-9.]\+\).*$/\1/' "$target"/etc/redhat-release)"
fi

if [ -z "$version" ]; then
    echo >&2 "warning: cannot autodetect OS version, using '$name' as tag"
    version=$name
fi

# 打包成镜像， 镜像名是$name, tag是$version
tar --numeric-owner -c -C "$target" . | docker import - $name:$version

# 运行新建立的镜像
docker run -i -t $name:$version echo success

rm -rf "$target"
```

## 发布镜像

将镜像提交到镜像中心

```sh
$ docker push 本地的镜像名:镜像标签
```

## 镜像的本地存放

[Docker 镜像管理（一）：本地镜像、本地容器的文件存放目录](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2020/02/08/docker-image-manager.html)

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://www.lijiaocn.com/%E9%97%AE%E9%A2%98/2017/03/22/docker-search-registry.html "docker搜索其它registry中的镜像]"

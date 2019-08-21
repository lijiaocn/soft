<!-- toc -->
# Mac/macOS 使用手册 —— 网络指令

## netstat

macOS 上的 **netstat** 命令用来查看本机的网络连接情况，但是要注意，macOS 的 netstat 命令用法与 linux 上的 netstat 不同。譬如在 linux 中 `-p` 参数的意思是显示关联的进程的程序名，在 macOS 中是指定协议：

```sh
-p protocol
           Show statistics about protocol, which is either a well-known name for a protocol or an alias for it.
```

### 查看监听的端口和连接

查看 tcp 监听端口，在 linux 上是 netstat -lnt，在 macOS 中是：

```sh
$ netstat -n -a -p tcp |grep "LISTEN"

# -n:     含义与 linux 相同，显示数字
# -a:     显示所有 socket，带有这个参数，才能显示监听端口
# -p tcp: 指定 tcp 协议
#
# 最后用 grep 将监听状态的 socket 过滤出来
```

如下：

```sh
$ netstat -n -a -p tcp  |grep "LISTEN"
tcp46      0      0  *.5002                 *.*                    LISTEN
tcp4       0      0  127.0.0.1.51526        *.*                    LISTEN
tcp4       0      0  127.0.0.1.63886        *.*                    LISTEN
tcp46      0      0  *.80                   *.*                    LISTEN
...
```

## 查找监听端口的进程

在 linux 上 netstat 的 -p 参数会显示连接或者 socket 所属的进程号和程序名称，macOS 的 netstat 没有类似的选项，需要用其它方法找到监听端口的进程。 **lsof** 是最好的选择之一，在 macOS 上的用法和在 linux 中的用法相同：

```sh
# 查找监听 80 端口的进程
$ lsof -n -i :80 |grep LISTEN
com.docke  6777 lijiao   35u  IPv6 0x65955d0d6aba74bb      0t0  TCP *:http (LISTEN)

# -i，指定网络地址
```

[使用 lsof 代替 Mac OS X 中的 netstat 查看占用端口的程序](https://tonydeng.github.io/2016/07/07/use-lsof-to-replace-netstat/)

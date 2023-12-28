<!-- toc -->
# Perf 基本用法

Perf 有五个子命令，可以用 man 查看每个子命令的用法，例如 `man perf-stat`：

```sh
perf-stat(1), perf-top(1), perf-record(1), perf-report(1), perf-list(1)
```

perf 既可以获取新起的进程的事件统计，也可以获取已经存在的进程或线程的事件统计，后者用 `-p`、`-t` 指定进程号、线程号。

## perf stat 执行命令并记录它的事件

运行一个命令，并记录该命令的事件信息：

```sh
perf stat [-e <EVENT> | --event=EVENT] [-a] <command>
perf stat [-e <EVENT> | --event=EVENT] [-a] — <command> [<options>]
```

## perf top 查看特定事件的分布情况

perf top 实时显示事件在每个进程上的分布情况：

```sh
perf top [-e <EVENT> | --event=EVENT] [<options>]
```

## perf record 将事件信息保存到文件

文件名是 perf.data：

```sh
perf record [-e <EVENT> | --event=EVENT] [-l] [-a] <command>
perf record [-e <EVENT> | --event=EVENT] [-l] [-a] — <command> [<options>]
```

## perf report 读取文件

读取用 perf record 生成的文件：

```sh
perf report [-i <file> | --input=file]
```

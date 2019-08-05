<!-- toc -->
# 内存缓存

## buffer与cache

free的命令中有一栏是buff/cache，buffer和cache都是内存缓存，但是又有区别。

```sh
$ free
              total        used        free      shared  buff/cache   available
Mem:       32731868     1170284    28024100      254420     3537484    30824636
Swap:      67108860     3376124    63732736
```

buffer中缓存的是将要写到磁盘的数据，通过合并写的方式提高写入效率。

cache中缓存的是从磁盘读取的数据。

## 缓存命中率

BCC中的cachestat和cachetop分别用来查看整个系统的缓存读写命中情况、每个进程的缓存命中情况。

```sh
$ cachestat 1 3
   TOTAL   MISSES     HITS  DIRTIES   BUFFERS_MB  CACHED_MB
       2        0        2        1           17        279
       2        0        2        1           17        279
       2        0        2        1           17        279 

```

TOTAL是总的IO次数，DIRTIES是新增到缓存中的脏页。 

```sh
$ cachetop
11:58:50 Buffers MB: 258 / Cached MB: 347 / Sort: HITS / Order: ascending
PID      UID      CMD              HITS     MISSES   DIRTIES  READ_HIT%  WRITE_HIT%
   13029 root     python                  1        0        0     100.0%       0.0%
```

## 文件的缓存大小

pcstat用来查看一个文件的缓存大小：

```sh
$ pcstat /bin/ls
+---------+----------------+------------+-----------+---------+
| Name    | Size (bytes)   | Pages      | Cached    | Percent |
|---------+----------------+------------+-----------+---------|
| /bin/ls | 133792         | 33         | 0         | 000.000 |
+---------+----------------+------------+-----------+---------+
```

pcstat的安装方法：

```sh
$ go get golang.org/x/sys/unix
$ go get github.com/tobert/pcstat/pcstat
```

在测试文件读写性能的时候，要清理缓存：

```sh
$ echo 3 > /proc/sys/vm/drop_caches
```


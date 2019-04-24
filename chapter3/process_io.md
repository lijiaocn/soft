# 进程的IO状态

## pidstat查看进程的I/O状态

`-d`表示展示IO信息，1表示每秒输出一次：

```bash
$ pidstat -d 1 
13:39:51      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s     iodelay  Command 
13:39:52      102       916      0.00      4.00      0.00           0  rsyslogd
                               每秒读    每秒写  每秒取消的写 I/O延迟
                               单位KB    单位KB  单位KB       单位时钟周期
```

如果要查看具体某个进程，使用-p指定进程号，下面间隔 1 秒输出 3 组数据：

```sh
$ pidstat -d -p 4344 1 3
06:38:50      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
06:38:51        0      4344      0.00      0.00      0.00       0  app
06:38:52        0      4344      0.00      0.00      0.00       0  app
06:38:53        0      4344      0.00      0.00      0.00       0  app
```


## iotop查看进程I/O排行

iotop动态显示每个线程的IO操作情况，由高到底排序：

```bash
Total DISK READ :	0.00 B/s | Total DISK WRITE :      11.28 K/s
Actual DISK READ:	0.00 B/s | Actual DISK WRITE:       0.00 B/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
  329 be/4 nobody      0.00 B/s    3.76 K/s  0.00 %  0.00 % nginx: worker process
  446 be/4 root        0.00 B/s    7.52 K/s  0.00 %  0.00 % systemd-journald
12800 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % dockerd
    1 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % systemd --switched-root --system --deserialize 21
    2 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kthreadd]
    3 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [rcu_gp]
```

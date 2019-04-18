# 进程的IO状态

## pidstat查看进程的I/O状态

```bash
$ pidstat -d 1 
13:39:51      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s     iodelay  Command 
13:39:52      102       916      0.00      4.00      0.00           0  rsyslogd
                               每秒读    每秒写  每秒取消的写 I/O延迟
                               单位KB    单位KB  单位KB       单位时钟周期
```

## iotop查看进程I/O排行

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

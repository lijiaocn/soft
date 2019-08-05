<!-- toc -->
# 查看Linux文件系统状态

## 可用空间

可用空间，用`df`查看，可用inode，用`df -i`查看。

```sh
# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G  4.0K  3.9G   1% /dev/shm
tmpfs           3.9G  452K  3.9G   1% /run
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vda1        20G   14G  6.8G  67% /
/dev/vdb        100G   64G   37G  64% /data
tmpfs           798M     0  798M   0% /run/user/0
```

```sh
# df -ih
Filesystem     Inodes IUsed IFree IUse% Mounted on
devtmpfs         995K   360  994K    1% /dev
tmpfs            998K     2  998K    1% /dev/shm
tmpfs            998K   447  997K    1% /run
tmpfs            998K    17  998K    1% /sys/fs/cgroup
/dev/vda1         20M  231K   20M    2% /
/dev/vdb          50M  1.8M   49M    4% /data
tmpfs            998K     1  998K    1% /run/user/0
```

## 页面缓存和slab缓存

页缓存和可回收的slab缓存：

```sh
$ cat /proc/meminfo | grep -E "SReclaimable|Cached" 
Cached:           748316 kB     #页缓存，缓存磁盘的上的文件，不包括swap
SwapCached:            0 kB     #已经被从swap召回，但是还在swap文件中的内存页
SReclaimable:     179508 kB     #可回收的slab
```

## inode和文件目录项的缓存情况

inode和文件目录项的缓存情况：

```sh
$ cat /proc/slabinfo | grep -E '^#|dentry|inode' 
# name            <active_objs> <num_objs> <objsize> <objperslab> <pagesperslab> : tunables <limit> <batchcount> <sharedfactor> : slabdata <active_slabs> <num_slabs> <sharedavail> 
xfs_inode              0      0    960   17    4 : tunables    0    0    0 : slabdata      0      0      0 
... 
ext4_inode_cache   32104  34590   1088   15    4 : tunables    0    0    0 : slabdata   2306   2306      0hugetlbfs_inode_cache     13     13    624   13    2 : tunables    0    0    0 : slabdata      1      1      0 
sock_inode_cache    1190   1242    704   23    4 : tunables    0    0    0 : slabdata     54     54      0 
shmem_inode_cache   1622   2139    712   23    4 : tunables    0    0    0 : slabdata     93     93      0 
proc_inode_cache    3560   4080    680   12    2 : tunables    0    0    0 : slabdata    340    340      0 
# vfs索引节点缓存
inode_cache        25172  25818    608   13    2 : tunables    0    0    0 : slabdata   1986   1986      0 
# dentry是目录项
dentry             76050 121296    192   21    1 : tunables    0    0    0 : slabdata   5776   5776      0 
```

具体含义见`man slabinfo`，slabtop可以更好的展现slab状态：

```sh
Active / Total Objects (% used)    : 743718 / 794177 (93.6%)
Active / Total Slabs (% used)      : 26434 / 26434 (100.0%)
Active / Total Caches (% used)     : 103 / 138 (74.6%)
Active / Total Size (% used)       : 179223.89K / 191460.80K (93.6%)
Minimum / Average / Maximum Object : 0.01K / 0.24K / 8.00K

OBJS ACTIVE  USE OBJ SIZE  SLABS OBJ/SLAB CACHE SIZE NAME
211260 205043  97%    0.19K  10060	 21     40240K dentry
 49344  48896  99%    0.06K    771	 64	 3084K anon_vma_chain
 42752  42752 100%    0.02K    167	256	  668K kmalloc-16
 38656  30327  78%    0.03K    302	128	 1208K kmalloc-32
 36352  29070  79%    0.01K     71	512	  284K kmalloc-8
...
```

## 磁盘分区

用`parted`查看磁盘分区的文件系统：

```bash
$ parted /dev/vdb
GNU Parted 3.1
Using /dev/vdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 107GB
Sector size (logical/physical): 512B/512B
Partition Table: loop
Disk Flags:

Number  Start  End    Size   File system  Flags
 1      0.00B  107GB  107GB  xfs

(parted)
```

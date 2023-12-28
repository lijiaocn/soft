# 进程的分类

[Understanding Linux Process States](https://access.redhat.com/sites/default/files/attachments/processstates_20120831.pdf)对Linux的进程（Process）做了很详细的介绍。

在Linux中，进程是申请资源的最小单位，每个进程有`ownership`、`nice value`、`SELinux context`等诸多属性。Linux中，进程分为三类：用户进程（User Process）、守护进程（Daemon Process）、内核进程（Kernel Process）。

`用户进程（User Process）`是普通用户创建的、运行在用户态（User Space）的进程。除非进行了特殊的权限设置，否则用户进程没有其它用户文件的权限。

`守护进程（Daemon Process）`是在后台一直运行的进程，应当算是用户进程中的一种，但它不随着用户的退出而终止，而是一直在系统中运行，直到被人为关闭。

`内核进程（Kernel Process）`只在内核态运行，它也是常驻的进程，但内核进程拥有最高权限，可以访问所有的内核数据。

这是通常提到的三种划分方法，个人感觉这种划分方法在标准上不统一，严格说，按照进程的所属和权限，进程可以分为用户进程和内核进程，按照进程的形态，可以分为守护进程和非守护进程。用户进程既可以是守护进程也可以是非守护进程，内核进程都是守护进程。

## 参考

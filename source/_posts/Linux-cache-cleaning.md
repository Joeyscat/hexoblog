---
title: Linux cache cleaning
date: 2018-03-16 21:27:53
categories:
- 笔记
tags: 
- Linux
---
### Linux下的缓存机制及清理buffer/cache/swap的方法梳理【转】

* 原文：[linux下的缓存机制及清理buffer/cache/swap的方法梳理](https://www.cnblogs.com/kevingrace/p/5991604.html)

#### 1.缓存机制介绍

在Linux系统中，为了提高文件系统性能，内核利用一部分物理内存分配出缓冲区，用于缓存系统操作和数据文件，当内核收到读写的请求时，内核先去缓存区找是否有请求的数据，有就直接返回，如果没有则通过驱动程序直接操作磁盘。
缓存机制优点：减少系统调用次数，降低CPU上下文切换和磁盘访问频率。
CPU上下文切换：CPU给每个进程一定的服务时间，当时间片用完后，内核从正在运行的进程中收回处理器，同时把进程当前运行状态保存下来，然后加载下一个任务，这个过程叫做上下文切换。实质上就是被终止运行进程与待运行进程的进程切换。

#### 2.查看缓存区及内存使用情况

```bash
joey@Joey:~$ top
top - 16:59:44 up  7:55,  1 user,  load average: 0.60, 0.73, 0.87
Tasks: 225 total,   1 running, 224 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.8 us,  0.5 sy,  0.0 ni, 97.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  8069972 total,  5060444 free,  1776368 used,  1233160 buff/cache
KiB Swap:  8000508 total,  8000508 free,        0 used.  5590520 avail Mem  
```

空闲内存=free + buffers + cached；
已用内存=total - 空闲内存；

#### 3.缓存区分buffers和cached区别

内核在保证系统能正常使用物理内存和数据量读写情况下来分配缓冲区大小。
buffers用来缓存metadata及pages，可以理解为系统缓存，例如，vi打开一个文件。
cached是用来给文件做缓存，可以理解为数据块缓存，例如，dd if=/dev/zero of=/tmp/test count=1 bs=1G 测试写入一个文件，就会被缓存到缓冲区中，当下一次再执行这个测试命令时，写入速度会明显很快。

#### 4.Swap用途

Swap意思是交换分区，通常我们说的虚拟内存，是从硬盘中划分出的一个分区。当物理内存不够用的时候，内核就会释放缓存区（buffers/cache）里一些长时间不用的程序，然后将这些程序临时放到Swap中，也就是说如果物理内存和缓存区内存不够用的时候，才会用到Swap。
swap清理：
swapoff -a && swapon -a
注意：这样清理有个前提条件，空闲的内存必须比已经使用的swap空间大

#### 5.释放缓存区内存的方法

1. 清理pagecache（页面缓存）

   ```bash
   echo 1 > /proc/sys/vm/drop_caches     或者 # sysctl -w vm.drop_caches=1
   ```

2. 清理dentries（目录缓存）和inodes

   ```bash
   echo 2 > /proc/sys/vm/drop_caches     或者 # sysctl -w vm.drop_caches=2
   ```

3. 清理pagecache、dentries和inodes

   ```bash
   echo 3 > /proc/sys/vm/drop_caches     或者 # sysctl -w vm.drop_caches=3
   ```

上面三种方式都是临时释放缓存的方法，要想永久释放缓存，需要在/etc/sysctl.conf文件中配置：vm.drop_caches=1/2/3，然后sysctl -p生效即可！

   另外，可以使用sync命令来清理文件系统缓存，还会清理僵尸(zombie)对象和它们占用的内存

--------------------友情提示一下----------------------
上面操作在大多数情况下都不会对系统造成伤害，只会有助于释放不用的内存。
但是如果在执行这些操作时正在写数据，那么实际上在数据到达磁盘之前就将它从文件缓存中清除掉了，这可能会造成很不好的影响。那么如果避免这种事情发生呢？
因此，这里不得不提一下/proc/sys/vm/vfs_cache_pressure这个文件，告诉内核，当清理inoe/dentry缓存时应该用什么样的优先级。

>vfs_cache_pressure=100    这个是默认值，内核会尝试重新声明
>dentries和inodes，并采用一种相对于页面缓存和交换缓存比较”合理”的比例。
>减少vfs_cache_pressure的值，会导致内核倾向于保留dentry和inode缓存。
>增加vfs_cache_pressure的值，（即超过100时），则会导致内核倾向于重新声明dentries和inodes
>总之，vfs_cache_pressure的值：
>小于100的值不会导致缓存的大量减少
>超过100的值则会告诉内核你希望以高优先级来清理缓存。
>其实无论vfs_cache_pressure的值采用什么值，内核清理缓存的速度都是比较低的。
>如果将此值设置为10000，系统将会将缓存减少到一个合理的水平。


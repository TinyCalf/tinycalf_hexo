---
title: Linux命令ulimit
date: 2019-09-17 11:42:24
tags:
- Linux
categories: Linux
---
> Linux中，ulimit用来限制当前用户可使用的各种系统资源，在开发中了解ulimit的内容还是相当重要的，这里就简单整理总结一下。

# ulimit用法
```bash
ulimit [-SHcdefilmnpqrstuvx] [limit]
```
我们经常还会用到开放所有限制来调试：
```bash
ulimit unlimited
```
<!-- more -->
# 所有资源限制类型
用ulimit查看当前所有资源
```bash
> ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15065
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15065
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
这样就显示了所有可以分配的资源，以下是逐个解释：
* **core file size**: 程序发生错误时会生成coredump文件以用来排除错误，这里用来限制coredump文件的大小；默认为0，所以默认情况下不生成coredump文件；这个选项非常重要，因为在prod环境部署的项目一定要开启这个限制，否则无法排除某些特殊的错误。
* data seg size: 每个进程数据段的最大值；数据段是用来初始化全局变量的一块区域，属于静态内存分配。
* scheduling priority: 进程优先级NICE值限制；改设置只对普通用户有效，限制普通用户进程可以设置的优先级范围。
* **file size**: 可创建最大文件大小。
* pending signals: 表示可以被挂起/阻塞的最大信号数量；
* max locked memory: 内存锁定值的限制；内存中的内容不不是一直在内存中的，有的时候再交换区或者磁盘上，所以有些时候我们希望有些数据只存在于内存里而不会转移到交换区或者磁盘；这里限制的是可以锁定到内存的数据的大小；
* max memory size: 最大内存限制；字面意思；但是在很多系统里没有作用；
* **open files**: 每个进程可以打开的文件数量；
* pipe size: 管道缓存；但是不能改变，只能是8 * 512(bytes)；
* POSIX message queues: 可以创建使用POSIX消息队列的最大值；
* real-time priority: 限制程序实施优先级的范围；
* **stack size**: 堆栈的最大值；
* cpu time: 每个进程可以使用CPU的最大时间;
* max user processes: 每个用户运行的最大进程并发数;
* virtual memory: 可使用的最大虚拟内存;
* file locks: 文件锁的限制;只在2.4内核之前有用;

# 软限制和硬限制
### 1.ulimit中有`-S` 和 `-H` 两个标记用来区分软限制和硬限制，无论是查看还是修改
比如要设置创建文件的大小的软上限，不指定SH的时候，同时设定软上限和硬上限；
```bash
ulimit -f 100 
```
然后我们用SH分别查看，可以看到软硬上限被同时设置了
```bash
$ ulimit -Sf
100
$ ulimit -Hf
100
```
### 2.软限制不能超过硬限制，硬限制也不能小于软限制
接着刚才，如果我们单独设置软上限超过硬上限或者硬上限小于软上限，就会报错：
```bash
$ ulimit -Sf 200
-bash: ulimit: file size: 无法修改 limit 值: 无效的参数
$ ulimit -Hf 10
-bash: ulimit: file size: 无法修改 limit 值: 无效的参数
```
### 3.普通用户只能缩小硬限制，超级用户可以扩大硬限制
普通用户缩小硬限制：
```bash
ulimit -Hf 50
-bash: ulimit: file size: 无法修改 limit 值: 无效的参数
```
### 4.硬限制的作用是控制软限制，软限制的作用是限制实际用户的使用
# ulimit设置永久生效
上面的命令只能临时修改ulimit的限制，修改修改/etc/security/limits.conf可以使设置永久生效。打开该文件有注释，注释的内容就是如何编写设置，比如我们要root用户文件大小限制都设置成4096，那么就在这个文件最下面添加以下内容：
```bash
root soft fsize 4096
root hard fsize 4096
```
下次登录时查看，就已经生效了：
```bash
$ ulimit -f
10000
```



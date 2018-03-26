---
title: Linux创建新用户
date: 2017-07-15 12:10:45
tags:
- Linux
categories: Linux
---
*  **创建用户**
```
$ useradd -r -m YOURNAME
```
* **设置密码** 一定要用这种形式，接着输入一下密码，不然会显示密码无效
```
$ passwd YOURNAME
```
结果是这样的
```
$ sudo passwd YOURNAME
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```
*  **用新用户登录**
```
$ sudo su - YOURNAME
```
然后试试`sudo`命令，发现不能用：
```
[sudo] password for YOURNAME:
YOURNAME is not in the sudoers file.  This incident will be reported.
```
不要紧我们先 `exit` 回到root，执行：
```
$ sudo usermod -a -G sudo YOURNAME
```
**注意：千万不要学网上找的教程说要改 /etc/sudoers<br>
注意：千万不要学网上找的教程说要改 /etc/sudoers<br>
注意：千万不要学网上找的教程说要改 /etc/sudoers**

*  最后测试以下新用户的 `sudo` 即可

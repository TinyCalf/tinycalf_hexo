---
title: Linux服务器搭建shadowsocks
date: 2018-03-21 17:19:13
tags:
- Linux
categories: Linux
author: TinyCalf
---

> 有的时候像npm apt之类的包管理工具连接不到资源会比较蛋疼。更关键的是程序出了bug不能google怎么办！！！来我们自己做个代理

### 服务器端搭建
我用的是ubuntu服务器，先安装Shadowsocks服务器端
```bash
apt-get update
apt-get install -y python-pip
pip install shadowsocks
```
<!-- more -->
此时系统会多出来两个程序：
```bash
/usr/bin/ssserver
/usr/bin/sslocal
```
新建一个注册文件，比如ssconfig.json:
```javascript
{
    "server":"0.0.0.0",
    "server_port":1999,
    "local_port":8989,
    "password":"password",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
然后启动sssserver服务

```
ssserver -c ./ssconfig.json --log-file /tmp/ss.log -d start
```
查看ss.log中的记录，确定是否启动成功
```bash
tail -f /tmp/ss.log
```
配置开机自动启动 修改 /etc/rc.local,在exit 0 之前添加以上代码，注意要用绝对路径
```bash
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

ssserver -c /root/ssconfig.json --log-file /tmp/ss.log -d start

exit 0
```
服务器设置完毕

### 客户端
## ubuntu/debian
安装shadowsocks-libev
```bash
sudo apt install shadowsocks-libev
```
同样要写一个配置文件，不同的是这回ip得改成刚才配置的那台服务器的，写在容易找到的文件夹里面
```
{
    "server":"the server ip",
    "server_port":1999,
    "local_port":8989,
    "password":"password",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
注意有两个端口，server_port是我们刚才配置服务器的ip，local_port是本地ip，也就是本地的各种应用想连vpn要通过的是本地的端口;然后我们开始运行本地ss服务
```bash
ss-local -c /path/to/your/ssconfig
```
后台运行的话这样：
```bash
nohup ss-local -c /home/tinycalf/ss/ssconfig &
```
设置开机自动启动和服务器一样，修改/etc/rc.local
```
ss-local -c /home/tinycalf/ss/ssconfig > /home/tinycalf/ss/out.log
exit 0
```
如果没有rc.local,直接把以上脚本重新命名，比如sslocal,放到 /etc/init.d下面，再链接到/etc/rc0.d/中即可
```
cd /etc/rc0.d
ln sudo ln -s ../init.d/ss-local ./ss-local
```
其他平台的基本都有界面化工具，输入你的服务器信息就可以了。

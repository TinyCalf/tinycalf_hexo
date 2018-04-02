---
title: 在Ubuntu上配置iptables
date: 2017-08-04 20:12:11
tags:
- Linux
- iptables
categories: Linux
---
# 列出当前规则
Ubuntu服务器默认不执行任何限制，但是为了将来参考，请使用以下命令检查当前的iptable规则
```
sudo iptables -L
```
<!-- more -->
# 添加规则
比如允许8888端口的访问：
```
sudo iptables -A INPUT -p tcp -m tcp --dport 8888 -j ACCEPT
```
# 保存和恢复规则
现在，如果要重新启动云服务器，那么所有这些iptables配置将被擦除。为了防止这种情况，请将规则保存到文件中
```
sudo iptables-save> /etc/iptables/rules.v4
```
然后，您可以通过阅读保存的文件来简单地还原保存的规则
```
＃覆盖当前规则
sudo iptables-restore </etc/iptables/rules.v4
＃添加新的规则保留当前的规则
sudo iptables-restore -n </etc/iptables/rules.v4
```
您可以在重新启动时自动执行恢复过程，方法是安装iptables的附加软件包，以接管已保存的规则的加载。使用以下命令给它
```
sudo apt-get install iptables-persistent
```
安装完成后，初始设置将要求保存IPv4和IPv6的当前规则，只需选择是并按两次。

如果您进一步更改iptables规则，请记住使用与上述相同的命令再次保存它们。iptables持久性在/ etc / iptables下查找文件rules.v4和rules.v6。

这些只是您可以使用iptables的几个简单命令，它能够提供更多的功能。请阅读以检查可用于更高级控制iptable规则的其他一些选项。

#参考地址
https://www.upcloud.com/support/configuring-iptables-on-ubuntu-14-04/

---
title: Windows 发布 bitcoin-qt
date: 2017-09-04 23:11:54
tags:
- 区块链
- 比特币
- BitcoinCore
categories: 区块链
thumbnail: http://ow3lvmu74.bkt.clouddn.com/images/WindowsBackground.jpg
author: Jonathan
---

> 本篇简单记录一下使用交叉编译发布Windows安装程序的方法

# 版本和环境
目前我在以下环境编译没有出现任何问题：

* Windows 10 64位 （推荐使用64位）
* Ubuntu 14.04.5 desktop amd64 虚拟机（14版ubuntu编译windows是最不容易出现问题的）
* Bitcoin Core v0.14.0(旧版本在出了linux的平台上发布问题比较多，因此这里用了比较新的版本)
<!-- more -->

# 交叉编译
## Unix环境搭建
首先按照doc/build_unix.md中的指示吧Unix的编译流程搭建好，这里不再多介绍了
## 交叉编译环境搭建
### 安装所需环境：
```bash
sudo apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils curl
```
#### 安装64位下的环境：
```bash
sudo apt-get install g++-mingw-w64-x86-64 mingw-w64-x86-64-dev
```
使用命令编译：
```bash
cd depends
make HOST=x86_64-w64-mingw32
cd ..
./autogen.sh # not required when building from tarball
CONFIG_SITE=$PWD/depends/x86_64-w64-mingw32/share/config.site ./configure --prefix=/
make
```
#### 如果你是32位：
```bash
sudo apt-get install g++-mingw-w64-i686 mingw-w64-i686-dev
```
使用命令编译：
```bash
cd depends
make HOST=i686-w64-mingw32
cd ..
./autogen.sh # not required when building from tarball
CONFIG_SITE=$PWD/depends/i686-w64-mingw32/share/config.site ./configure --prefix=/
make
```
### 发布安装程序
```bash
make install DESTDIR=/mnt/c/workspace/bitcoin
```
## 截图
![](/images/windows1.png)
![](/images/windows2.png)

---
title: Nodejs快速安装
date: 2017-07-20 23:11:54
tags:
- nodejs
- nvm
- npm
categories: nodejs
thumbnail: http://ow3lvmu74.bkt.clouddn.com/NodeJsBackground.jpg
author: TinyCalf
---

> 用安装包上传安装实在太慢了 <br>
> 而且多个版本的node也没法发控制 <br>
> 我们使用nvm工具会快的多

## 假设环境设定
*  ubuntu v16.04
*  没有安装npm和node
<!-- more -->
## 安装 npm
```bash
$ apt install npm
```
## 安装 nvm   
[找 nvm git 库点我](https://github.com/creationix/nvm) 里面也有安装说明<br>
执行：
```bash
$ wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```
再执行：
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```
输入 `nvm` 看看有没有正确安装

## 安装nodejs
假设我们要装 v4版本的node：
```bash
$ nvm install v4.*
```
如果还装过别的版本则执行一下来切换版本：
```bash
$ nvm use 4
```

## OK，安装好了，快吧～

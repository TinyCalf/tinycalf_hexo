---
title: Nodejs快速安装
date: 2017-07-20 23:11:54
tags:
- nodejs
- nvm
- npm
categories: nodejs
thumbnail: http://ow3lvmu74.bkt.clouddn.com/NodeJsBackground.jpg
author: Jonathan
---



## VIM 使用基础

```bash
:set nu
```

设置行号显示
```bash
:set number
```

tab等于多少空格
```bash
:set tabstop=2
```

代码高亮
```bash
:syntax on
```

自动缩进
```bash
:set autoindent
```

你也可以使用配置文件设定这些参数，在每次打开的时候自动应用这些设置
```bash
cd ~
vi .vimrc
```
然后将以上设置写进去
```bash
set nu
set tabstop=2
syntax on
set autoindent
```
然后 `:wq` 退出

在vim界面中可以查看使用的配置文件，输入
```
:echo $MYVIMRC
```
即可显示当前vim配置文件路径

V模式下拖黑文字
D键为删除


```bash
#!/bin/bash

# Comment

echo "Hello World!"
```

给该文件执行权限

```bash
sudo chmod +x shelltest
```

然后就可以执行该文件了

```bash
./shelltest
```

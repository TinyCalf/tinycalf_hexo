---
title: 解决Telegram For Linux输入法问题
date: 2017-07-23 18:10:45
tags:
- Linux
categories: Linux
---

> 虽然Telegram新版本已经解决这个问题，但是在我的debian上还是出现了无法切换输入法的问题。以下是解决方案

打开desktop文件
```bash
sudo vi /usr/share/applications/telegramdesktop.desktop
```
如果你的路径和我不一样就locate一下
```bash
locate telegramdesktop.desktop
```
<!-- more -->
把Exec那一行修改一下，如下
```bash
Exec=env QT_IM_MODULE=fcitx /usr/bin/telegram-desktop -- %u
```
其中`/usr/bin/telegram-desktop`的路径也有可能和我不一样，使用whereis搜索一下telegram的启动程序：
```bash
whereis telegram-desktop
```
好了，保存即可，重启一下telegram。

---
title: Linux中使用grep在文件夹中搜索文件内容
date: 2019-03-30 08:52:51
tag:
- Linux
- 小技巧
categories: Linux
author: TinyCalf
---

```bash
grep -rnw '/path/to/somewhere' -e 'pattern'
```
* `-r` 表示递归后面的路径
* `-n` 表示是否显示所有文件行数
* `-w` 表示完全匹配
* `-l` 可以显示文件内容
* `-e` 后面的pattern是要搜索的内容，可以是正则表达式
<!-- more -->
例如我在我的博客代码里搜索类型为Linux的文章：
```bash
$ grep -rnw ./ -e 'categories:.*Linux'
.//source/_posts/1000-Linux中使用grep在文件夹中搜索文件内容.md:7:categories: Linux
.//source/_posts/在Ubuntu上配置iptables.md:7:categories: Linux
.//source/_posts/siege（围攻）web压力测试工具.md:7:categories: Linux
.//source/_posts/搭建shadowsocks.md:6:categories: Linux
.//source/_posts/node+nginx搭建SSL.md:8:categories: Linux
.//source/_posts/Linux创建新用户.md:6:categories: Linux
```
还有一个比较有用的flag: `--include='pattern'`,可以用来搜索指定名称的文件，比如：
```bash
$ grep -rnw ./ -e 'categories:.*Linux' --include='*.md'
```
`grep`是非常常用的命令，可以`man grep`获取更多相关信息。
如果搜索这个需求比较频繁可以做成全局的小工具fif(Find in Folder)：
```
sudo echo grep -rnw \`pwd\` -e \$1 > /usr/local/bin/fif && chmod +x /usr/local/bin/fif
```
这样用：
```bash
fif 'categories:*Linux'
```

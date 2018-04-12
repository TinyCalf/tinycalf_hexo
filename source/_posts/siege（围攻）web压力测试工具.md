---
title: siege（围攻）web压力测试工具
date: 2018-04-12 17:48:57
tag:
- Linux
- Stress Test
categories: Linux
author: TinyCalf
---

> Siege是一个轻量级web压力测试脚本，支持POST和GET,能对多个api进行持续性的测试，操作过程简便


## 官网
https://www.joedog.org/

## 安装
* 使用包管理工具安装，比如apt
```bash
sudo apt install siege
```

<!-- more -->
* 编译安装
```bash
wget http://www.joedog.org/pub/siege/siege-2.70.tar.gz
tar zxvf siege-2.70.tar.gz
./configure
make
sudo make install
```

## 常用参数
指定并发数量为100
```
-c 100
```
指定访问次数为10
```
-r 10
```
指定访问时间，单位分钟，和指定访问次数不能同时使用
```
-t 5
```
指定url文件,保存了所有需要测试的url，后面还会说详细用法
```
-f apis.txt
```
从上面的url文件里面随机选择api来测试
```
-i
```
指定HTTP头
```
-H "Content-Type:application/json"
```

## 用例
仅测试一个url
```
siege -c 50 -r 5 http://127.0.0.1:3000/v1/getusers
```
测试一个POST接口，注意最后一个参数是一整个字符串，包含了url和json
```
siege -H "Content-Type:application/json" -c 1 -r 1 'http://127.0.0.1:1990/v1/submit POST {"user":"tinycalf","pass":"123321"}'
```
使用文件保存所有url，我们创建一个文件apis，一行一个请求，就像这样
```
http://127.0.0.1:3000/v1/getusers
http://127.0.0.1:1990/v1/submit POST {"user":"tinycalf","pass":"123321"}
```
然后如下开始测试
```
siege -c 50 -r 50 -f ./apis -i -b
```
就能得到测试结果
```
** SIEGE 4.0.4
** Preparing 50 concurrent users for battle.
The server is now under siege...
Transactions:		        2500 hits  
Availability:		      100.00 %
Elapsed time:		        0.89 secs
Data transferred:	        3.90 MB
Response time:		        0.02 secs
Transaction rate:	     2808.99 trans/sec
Throughput:		        4.38 MB/sec
Concurrency:		       48.91
Successful transactions:        2500
Failed transactions:	           0
Longest transaction:	        0.05
Shortest transaction:	        0.00
```

* Transactions: 总共测试次数
* Availability: 成功次数百分比
* Elapsed time: 总共耗时多少秒
* Data transferred: 总共数据传输
* Response time: 等到响应耗时
* Transaction rate: 平均每秒处理请求数
* Throughput: 吞吐率
* Concurrency: 最高并发
* Successful transactions: 成功的请求数
* Failed transactions: 失败的请求数

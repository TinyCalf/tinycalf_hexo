---
title: pm2功能概要
date: 2019-06-27 9:27:47
tags:
- nodejs
- pm2
categories: nodejs
author: TinyCalf
---

> pm2具体文件见官网，下面有链接；这里只是整理一下pm2的所有功能，用到的时候又个对照。


## 功能列表
* 多种配置文件形式；yml、json、js均支持；
* 应用启动脚本配置；简单的`node app.js` 和复杂的脚本启动均可配置；
* watch功能；检测文件变化并自动重启服务，在测试环境中非常常用；
* 以cluster为基础的多进程部署；instances、exec_mode；对于web应用来说相当于自动负载均衡；
* 可指定启动脚本执行的路径；也就是不用非得在项目目录下执行；
* 指定进程最大内存；max_memory_restart; 在内存超过该值时会重新启动；
* 日志相关；错误输出和其他输出是分开记录在文件中的，可改变默认日志路径；也可以指定是否将多个进程的日志集中在一起；
* 进程控制相关；指定启动时间、超时时间、自动重启等等；
* 可指定解释器；bash、python；但是目前我还没用过
* 可通过ssh方式远程发布（Deployment）；配置文件中可指定发布服务器信息；pm2命令和操作远程项目；
* 查看进程列表；
* 查看单独的进程日志或者多个进程的联合日志；
* 查看运行状态，cpu内存堆栈资源利用情况；
* 控制进程的启动暂停重置删除；
* 配置服务器重启后的自动恢复（Startup）；
* 支持安全退出进程；前提是代码中时通过process按照标准实现安全退出；
* 可配合keymetrics进行监控报警(Process Actions)；运维应该非常喜欢这个功能；
* 支持BabelJS， Typescript的source map，以追踪这些代码的错误源文件；
* pm2支持api；也就是说你可以通过api编写一个自己的管理软件；
* 管理node模块的发布和加载（Module System）；这个我还没试过；
* pm2可以和nginx一样作为静态文件的服务器；但是功能比较简单，测试用一用还是很方便的；
* 支持Docker

<!-- more -->




## CheatSheet
```bash
# Fork mode
pm2 start app.js --name my-api # Name process

# Cluster mode
pm2 start app.js -i 0        # Will start maximum processes with LB depending on available CPUs
pm2 start app.js -i max      # Same as above, but deprecated.
pm2 scale app +3             # Scales `app` up by 3 workers
pm2 scale app 2              # Scales `app` up or down to 2 workers total

# Listing

pm2 list               # Display all processes status
pm2 jlist              # Print process list in raw JSON
pm2 prettylist         # Print process list in beautified JSON

pm2 describe 0         # Display all informations about a specific process

pm2 monit              # Monitor all processes

# Logs

pm2 logs [--raw]       # Display all processes logs in streaming
pm2 flush              # Empty all log files
pm2 reloadLogs         # Reload all logs

# Actions

pm2 stop all           # Stop all processes
pm2 restart all        # Restart all processes

pm2 reload all         # Will 0s downtime reload (for NETWORKED apps)

pm2 stop 0             # Stop specific process id
pm2 restart 0          # Restart specific process id

pm2 delete 0           # Will remove process from pm2 list
pm2 delete all         # Will remove all processes from pm2 list

# Misc

pm2 reset <process>    # Reset meta data (restarted time...)
pm2 updatePM2          # Update in memory pm2
pm2 ping               # Ensure pm2 daemon has been launched
pm2 sendSignal SIGUSR2 my-app # Send system signal to script
pm2 start app.js --no-daemon
pm2 start app.js --no-vizion
pm2 start app.js --no-autorestart
```
## 相关网址
[pm2官网](http://pm2.keymetrics.io/)
[pm2文档](http://pm2.keymetrics.io/docs/usage/quick-start/)
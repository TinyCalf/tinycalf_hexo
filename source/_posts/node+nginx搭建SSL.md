---
title: node+nginx配置ssl证书
date: 2017-09-09 14:17:47
tags:
- nodejs
- nginx
- ssl
categories: Linux
thumbnail: http://ow3lvmu74.bkt.clouddn.com/image/postbackgroundHttpsBackground.jpg
author: Jonathan
---

> 本节记录一下node.js项目如何配合nginx配置ssl证书，主要用来解决copay的iOS客户端链接bws时出现的BADREQUEST问题，顺便优化一下url

## copay客户端问题
在我们之前修改好的copay项目已经链接到山寨币的情况下，启动iOS工程以后服务器端会出现如下报错：

```bash
BADREQUEST Required argument name missing
```

导致客户端无法创建钱包、无法生成地址。原因在copay的github讨论中有涉及到：<br>
https://github.com/bitpay/bitcore-wallet-service/issues/628
这个问题目前只出现在了ios上，原因是域名没有ssl证书，下载或者申请一个，用nginx解析一下即可

## 申请并下载ssl证书
阿里云可以申请免费的证书，或者去其他地方申请，下载后应该有两个文件。

```bash
xxxx.pem
xxxx.key
```

最好申请两个证书，一个给insight，一个给bws，并接解析两个域名，比如我的insight网址和bws接口地址：<br>
https://blockchain.browser.tiny-calf.com/insight/
https://bws.tiny-calf.com/bws/api
正常ssl证书可以分配多个域名的，并且可以使用通配符，但是阿里的免费证书只支持一个耳机域名，所以要申请两个。

## 安装配置nginx
### 安装

```bash
apt install nginx-core
```
### 上传证书
完成后应该自动启动了，然后进入 */etc/nginx/* 并创建一个文件夹把我们刚下载的证书放进去,比如我创建的是cert文件夹：
```bash
cd /etc/nginx/
mkdir cert
```
### 配置新的虚拟主机

回到上级目录我们查看以下 *nginx.conf* 其中引用的两个文件夹的配置文件
```bash
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```
说明我们需要在 *sites-enabled* 中添加虚拟主机，但是 *sites-enabled* 下实际上是 *sites-available* 中文件的链接，所以我们实际上实在 *sites-available* 中创建新的虚拟主机的配置文件，然后在把需要用的文件链接到 *sites-enabled* 。OK，那来吧。进入 *sites-enabled*
有个 *default* 文件，复制两个出来，分别命名为 *insight* 和 *bws*，然后两个文件的配置其实是差不多的，我拿insight文件作为例子：
```bash

server {
        listen 80;
        #替换成你的域名
        server_name blockchain.browser.tiny-calf.com;
        #将http请求强制转到https
        rewrite ^/(.*) https://blockchain.browser.tiny-calf.com/$1 permanent;
}

server {
        ssl on;
        #换成你的证书名称
        ssl_certificate   cert/xxxxxxxx.pem;
        ssl_certificate_key  cert/xxxxxxxx.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        index index.html index.htm index.nginx-debian.html;
        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_set_header X-NginX-Proxy true;
                #insight的nodejs端口为3001,因此这里吧请求全转到3001端口
                proxy_pass http://127.0.0.1:3001;

                proxy_redirect off;
        }
}

```

重要配置看注释，然后 *bws* 也是一样，*bws* 的端口号为3232,记得换一下。然后我们把 *bws* 和 *insight* 连接到 *sites-enabled* 并删除原来 *default* 的链接
```bash
cd sites-enabled
sudo rm default
sudo ln -s ../sites-available/insight ./
sudo ln -s ../sites-available/bws ./
```
然后重启nginx：

```bash
sudo /usr/sbin/nginx -s reload
```
项目的各个配置文件实际上不用改，因为用的都是 *localhost* ，这样的话直接运行节点，看看能不能正常访问：

![](/images/insight.png)

这样就说说明配置正确了，bws的话基本也没问题了，运行 *copay* 客户端看看是不是能正常使用了，记得搜索项目中所有带有 *bws/api* 的uri，都换成自己的好了。
## 完成
如果配置全都正确现在iOS客户端应该已经可以正常使用了

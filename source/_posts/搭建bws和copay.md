---
title: 搭建轻量级数字货币钱包
date: 2017-08-09 14:17:47
tags:
- 区块链
- 比特币
- BWS
- copay
categories: 区块链
thumbnail: /images/posts/wallet.jpg
---

> 本节我们在bitcore的基础上搭建我们自己货币的BWS（bitcore-wallet-service,比特核心钱包服务）以及依赖该服务的钱包客户端copay

## 准备条件

确保你已经成功编译和运行了 bitcoin core 正如我最早的几篇教程所讲，另外确保你已经成功搭建 bitcore 和区块链浏览器 insight ，因为我是接着那一篇来讲的，没有的话看这里：<br>

[创造数字货币（4）-- 搭建区块链浏览器](http://www.tiny-calf.com/chuang-zao-shu-zi-huo-bi-4-ji-yu-bitcorede-qu-kuai-lian-liu-lan-qi/)<br>

## 安装 *BWS*
之前我讲过用 *nohup* 启动守护进程，如果你也这么做了，那就先关闭正在运行的 *bitcored*

```bash
killall bitcored
```
*BWS* 使用 *mongodb* 作为数据库，安装 *mongodb*

```bash
sudo apt-get install mongodb
```

安装完成以后会自动启动 *mongdb* 可以用命令`$ mongo`试一下能不能进入mongo的shell界面：

```bash
$ mongo
MongoDB shell version: 2.6.10
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
>

```

可以的话就 `exit`。
然后 *BWS* 可以单独安装，但是我们既然做了节点，那么就可以用 *bitcore* 安装：
```bash
cd yournode
bitcore install bitcore-wallet-service
```

修改以下bws的配置文件，在节点目录下的这个位置：

```bash
./node_modules/bitcore-wallet-service/config.js
```

可以按照自己的改，但是如果你是按照我的教程做的话，只要改一个地方：

```javascript
blockchainExplorerOpts: {
    livenet: {
      provider: 'insight',
      url: 'https://insight.bitpay.com:443',
    },
    testnet: {
      provider: 'insight',
      url: 'https://test-insight.bitpay.com:443',
      // url: 'http://localhost:3001',
      // Multiple servers (in priority order)
      // url: ['http://a.b.c', 'https://test-insight.bitpay.com:443'],
    },分支
  },
```

把这里面的url统统换成你自己的insight地址，服务器地址或者localhost：

```javascript
blockchainExplorerOpts: {
  livenet: {
    provider: 'insight',
    url: 'http://localhost:3001/insight',
  },
  testnet: {
    provider: 'insight',
    url: 'http://localhost:3001/insight',
    // url: 'http://localhost:3001',
    // Multiple servers (in priority order)
    // url: ['http://a.b.c', 'https://test-insight.bitpay.com:443'],
  }
	...
}
```

保存退出，回到节点根目录，启动bitcored，然后bws的调试信息也会一块儿输出，很好区分，bitcore的输出是带时间信息的，bws的输出则没有时间信息。你可以看到bws有这些信息：

```bash
info Using message broker server at http://localhost:3380
info Using locker server:localhost:3231
ERR! Error connecting to Insight (livenet) @ http://localhost:3001
Connection established to mongoDB
info Limiting wallet creation per IP: 20 req/h
info Using locker server:localhost:3231
```

一开始会提示ERR！，这只是因为bws和bitcored是同时启动的，过几秒就会反馈链接成功的信息：

```bash
info Connected to Insight (testnet) @ https://test-insight.bitpay.com:443
info New livenet block: 000002a3846afeaba0f9fe8b8cbbb1708c1a6059c0aa991830384edcb8c6b1e9
```

看到这里基本上就安装成功了，但是具体有没有安装成功我们还是得搭建好copay客户端来验证一下
## 搭建 *copay*
copay支持所有平台的客户端，包括android/iOS/MacOS/Win/Web等，非常值得我们开发和研究，git地址在这，可以可上他们关网看看：<br>https://github.com/bitpay/copay<br>
github上关于安装讲的比较详细，我这只挑关键步骤，然后主要讲怎么连到我们的山寨币上
### 运行原版客户端
在又nodejs的情况下，两句命令就能直接在web上查看copay客户端：
```bash
npm run apply:copay
npm start
```
运行成功了就关掉做下一步
### 修改api地址
我们刚才搭建了bws，所以我们要把api往bws上面换，bws默认接口是 http://localhost:3232/bws/api<br>所以我们换成这个就行，怎么换呢，改这个文件：
```bash
/home/jonathan/copay/src/js/services/configService.js
```
内容是这样的
```javascript
bws: {
      url: 'https://bitpay.com/bws/api',
    },

    download: {
      bitpay: {
        url: 'https://bitpay.com/wallet'
      },
      copay: {
        url: 'https://copay.io/#download'
      }
    },

    rateApp: {
      bitpay: {
        ios: 'http://itunes.apple.com/WebObjects/MZStore.woa/wa/viewContentsUserReviews?id=1149581638&pageNumber=0&sortOrdering=2&type=Purple+Software&mt=8',
        android: 'https://play.google.com/store/apps/details?id=com.bitpay.wallet',
        wp: ''
      },
      copay: {
        ios: 'http://itunes.apple.com/WebObjects/MZStore.woa/wa/viewContentsUserReviews?id=951330296&pageNumber=0&sortOrdering=2&type=Purple+Software&mt=8',
        android: 'https://play.google.com/store/apps/details?id=com.bitpay.copay',
        wp: ''
      }
    },
```
把最上面那个url换成我们搭建好的bws的接口地址，如果要做成产品其他地址也得换，你们看着办就行。第一个url最重要，关系到能不能调通钱包。改好以后保存退出，做下一步。

### 修改网络参数

和上一篇一样，copay里面也有bitcore-lib模块，我们需要修改networks.js中的addNekwork参数，参照上一篇就行。networks.js的地址在copay的这里：

```bash
./node_modules/bitcore-wallet-client/node_modules/bitcore-lib/lib/networks.js
```

*注意：修改api时候那个文件configService.js，能通过git保存，但是networks.js在node模块中，是安装的时候下载的，没法通过git同步，这里建议把network.js考出来做的脚本，在别的地方安装的时候直接复制进去即可*

### 运行测试

用之前的方式启动copay，在浏览器中获取地址，从你搭建好的钱包打点钱过来，如果能打过来，就表示成功了！

![](/images/2017-07-20-10-50-44----.png)

## TODO
* 修改界面
* 修改各种url链接（可别连到别人那去）

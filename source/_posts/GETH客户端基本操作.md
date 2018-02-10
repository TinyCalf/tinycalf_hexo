---
title: GETH客户端基本操作
date: 2018-02-01 23:11:54
tags:
- 区块链
- 以太坊
- ETH
- ethereum
- go-ethereum
categories: 区块链
thumbnail: http://ow3lvmu74.bkt.clouddn.com/image/geth.jpg
author: Jonathan
---

> 我写的肯定没有文档全，但是文档太多了，刚看的童鞋肯定一脸蒙B，所以我整理了一下开发以来最常用到的操作，希望大家能少踩些坑

### 准备工作
* geth，全名go-ethereum，git项目地址：https://github.com/ethereum/go-ethereum
* 命令行的API可以参考这个 https://github.com/ethereum/go-ethereum/wiki/Management-APIs
* web3,即javascript的API，使用方法看 https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgetbalance
* geth下载地址 https://geth.ethereum.org/downloads/


### 启动geth
没错，客户端下载下来就一个geth文件，有三种方法可以启动，按自己的需要来
##### 带前端交互的启动
这种启动方式可以通过命令行直接控制钱包，但是退出进程后客户端也不会再后台运行
```bash
cd path/to/geth/
./geth console 2>>eth.log
```
效果应该是这样的
```bash
$./geth console 2>>eth.log
Welcome to the Geth JavaScript console!

instance: Geth/v1.7.2-stable-1db4ecdc/linux-amd64/go1.9
coinbase: 0x6847aee3f59652e3ddc51087416360f17ab9c8b1
at block: 0 (Thu, 01 Jan 1970 08:00:00 CST)
 datadir: /home/jonathan/Desktop/Coinnodes/ETH-1.7.2/data
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

>
```
`>`后面可以输入命令，这个我们后面会讲。当然启动的时候有一些比较叫重要的参数有必要讲一下，我常用的启动脚本是这样的：
```bash
./geth  --rpc --rpcaddr 127.0.0.1 --port 10071 --rpcport 10070 --rpcapi "personal,db,eth,net,web3" --datadir "./data" console 2>>eth.log
```
* `--rpc` 表示启用rpc端口
* `--rpcaddr` rpc端口的地址
* `--rpcport` rpc所在端口
* `--rpcapi` 规定rpc可访问的命令范围，有personal,db,eth,net,web3等，用逗号隔开
* `--port` geth节点p2p的端口
* `--datadir` 规定本地数据库，区块链存储，以及私钥等存放的地址，默认地址 `～/.ethereum`
* `--dev` 加上这个参数则会启动私有链

写成shell脚本效果更佳～

##### 前后端分离的启动
这种方式既可以随时和geth交互，又不影响geth在后台运行，在服务器上非常有必要。先用nohup在后台进程中跑geth
```bash
nohup ./geth --datadir "./data" &
```
然后使用attach命令开启交互界面
```bash
./geth attach ipc:./data/geth.ipc
```
geth启动时会在我们规定的data目录生成ipc，但是这个ipc因为版本的不同可能出现在不同的位置，找一下就好，我们把attach的地址设置成该ipc所在地址，即可再次进入交互界面，用`exit`退出也不会影响后台进程。

### 同步
**以下所有命令在geth的console中输入即可**
##### 查看同步状态
```javascript
eth.syncing
```
这个命令会返回false或者一个同步状态的数组;有两种情况会返回false，一是还没找到任何节点，并且没有开始同步，二是找到节点了并且已经同步到最高块;如果返回数组则表示正在同步，并输出最高块和当前同步的块高

##### 查看当前同步高度
```javascript
>eth.blockNumber
5220670
```

##### 查看节点数量
看看当前是不是已经连上其他节点了
```javascript
> net.peerCount
5
```

##### 查看当前同步节点的详细信息
```javascript
>admin.peers
[{
    caps: ["eth/62", "eth/63", "par/1", "par/2", "pip/1"],
    id: "3148c38ffb1b5328cfd121653cc70ec68a44390d68e304d3fa8b09fcf6a28e9b13e5125332ad8da194ce740eaf011daa6f36dfee43a3a2f43e067776928876ea",
    name: "Parity/v1.7.8-unstable-d5fcf3b-20171025/x86_64-linux-gnu/rustc1.19.0",
    network: {
      localAddress: "172.31.32.9:46758",
      remoteAddress: "45.56.113.50:30304"
    },
    protocols: {
      eth: {
        difficulty: 205043033534245860000,
        head: "0x5b1ac93fc624387d6805d350284ec5faa71f61a463babfd1086771085bf0a4b1",
        version: 63
      }
    }
}, {
    caps: ["eth/62", "eth/63", "par/1", "par/2", "pip/1"],
    id: "cef80e2721f25ae6468f5f3db3ed05a7ea6b3278b8c9acd1afc103508eaf3e2d56846eac5854b9e18bad99b61d1301850ca01e126df4df63804af899565f6570",
    name: "Parity/v1.8.2-unstable-1b6588c-20171025/x86_64-linux-gnu/rustc1.20.0",
    network: {
      localAddress: "172.31.32.9:20368",
      remoteAddress: "217.182.90.254:30303"
    },
    protocols: {
      eth: {
        difficulty: 205043033534245860000,
        head: "0x5b1ac93fc624387d6805d350284ec5faa71f61a463babfd1086771085bf0a4b1",
        version: 63
      }
    }
}]
```
##### 增加节点
以太坊经常会出现半天都找不到节点的情况，我们需要手动添加一些节点，比如
```javascript
>admin.addPeer("enode://20c9ad97c081d63397d7b685a412227a40e23c8bdc6688c6f37e97cfbc22d2b4d1db1510d8f61e6a8866ad7f0e17c02b14182d37ea7c3c8b9c2683aeb6b733a1@52.169.14.227:30303")
true
```
类似括号中的节点信息可以百度找一些。

### 账户
#### 创建账户
```javascript
personal.newAccount([你的密码])
```
#### 查看所有帐号
以太坊和比特币不同，比特币客户端只支持一个账户，一个账户下可生成无限地址;而以太坊能支持无限个账户，但是每个账户都只有一个地址，我们可以这样查看所有帐号：
```javascript
> eth.accounts
["0x6847aee3f59652e3ddc51087416360f17ab9c8b1"]
```
输出结果就是我们刚创建的帐号，如果我们创建更多,就会这样:
```javascript
> eth.accounts
["0x6847aee3f59652e3ddc51087416360f17ab9c8b1", "0xa9ee089a9a8837bfd0b232198e5850ba94ca4ab4"]
```
我们可以用javascript的语法获取其中某个地址
```javascript
> eth.accounts[0]
0x6847aee3f59652e3ddc51087416360f17ab9c8b1
```
#### 解锁帐号
所有转出的操作之前必须先解锁帐号。比如我们想解锁帐号0:
```javascript
> personal.unlockAccount(eth.accounts[0],"PASSWORD",1000000)
true
```
参数依次是 帐号/密码/时间（毫秒）

### 挖矿
在测试网络中我们需要挖矿来获取ether,另外要保证已经创建过帐号才能进行挖矿。
#### 开始
```javascript
miner.start(1)
```
括号里的数字表示进程数，不写会默认占用你所有cpu资源。挖矿得到的ether会自动打到第一个账户中。
#### 停止
```javascript
miner.stop()
```
结合上面的”查看当前同步高度“可以看看挖了多少

### 资金
#### 查看账户余额
我们在测试网中挖到矿以后，就可以查看下余额
```javascript
var ac = eth.accounts[0]
var balance = eth.getBalance(ac)
web3.fromWei(balance)
```
输出
```javascript
320
```
其中fromWei是web3提供的单位转换操作，关于类似操作下面还会提到
#### 转账
我们以测试网络中帐号0转100个ETH给帐号1为例子,需要结合上面的解锁账户功能：
```javascript
var from = eth.accounts[0]
var to = eth.accounts[1]
var amount = 100
personal.unlockAccount(from,"PASSWORD",100000)
eth.sendTransaction({
  from:from,
  to:to,
  value:web3.toWei(amount),
})
```
这个命令中我们只需要修改一开始定义的 from（发送帐号）/to（目标帐号）/amount（数量）即可;另外我们还是使用了web3.toWei来转换单位。
但是实际中使用会相对复杂一些，比如你需要提取一个账户中所有的余额，那你就需要计算一下需要多少矿工费，并扣除矿工费转出：
```javascript
var from = eth.accounts[0]
var to = eth.accounts[1]
personal.unlockAccount(from,"PASSWORD",100000)
tx={
    from:from,
    to:to,
    value:eth.getBalance(from)
}
var gasPrice=eth.gasPrice;
var gas=eth.estimateGas(tx);
tx.gasPrice=gasPrice;
tx.gas=gas;
tx.value=web3.toBigNumber(tx.value).minus(gasPrice*gas);
eth.sendTransaction(tx)
```
这个操作中我们先定义了tx，假设发送数量为所有余额，这样我们就能通过estimateGas来计算所需要的gas，实际的矿工费为gasPrice * gas(单位ether);
然后我们再在value中减去需要的矿工费，构成一笔交易单tx，使用sendTransaction将其广播。
这里面的加减法我们用了web3中的bignumber类，下面会详细说明;另外测试环境记得挖矿才能完成交易的操作

### 数学运算
#### fromWei 和 toWei
ether的最好单位是Wei，Wei不可分割，geth中所有关于金额的操作使用的单位都是Wei，于是就有了fromWei和toWei来转换单位;
比如我们getBalance获取的余额单位是Wei难以阅读：
```javascript
> web3.fromWei("123499999999999999800")
"123.4999999999999998"
```
或者
```javascript
> web3.fromWei("123499999999999999800","ether")
"123.4999999999999998"
```
再比如我们想发送100个ether，那transaction中的value就应该是：
```javascript
> web3.toWei("100")
"100000000000000000000"
```
#### bigNumber
ether最多可以有18位小数，js本身无法进行相应计算，比如我们在node或者geth中输入：
```javascript
> 123499999999999999800-0
123500000000000000000
```
明显发生了计算错误，所以就像我在转账中写的，**所有数学计算前都要转换成bignumber，并使用bignumber的计算方法计算**
```javascript
> web3.toBigNumber("123499999999999999800").minus(1)
123499999999999999799
```
记得toBigNumber中的参数是String

### 总结
我这里只写了geth客户端中最基本最常用的命令，为了读者们能快速上手。
由于geth客户端的交互使用的js脚本，因此操作非常灵活，我们可以根据自己的业务需求编写各种控制脚本。
想用nodejs通过外部RPC控制geth的话，你需要通过npm下载web3的模块;官方提供的web3中不包含personal/miner/admin，所以可能需要使用非官方的web3_extended。
以后可能会写个有关token的操作，因为那部分也比较复杂。

---
title: Solidity开发环境搭建及简单合约编写测试
date: 2018-03-16 14:17:47
tags:
- geth
- Ethereum
- Blockchain
categories: Solidity
author: TinyCalf
---

> 在学习solidity之前，我们先了解下开发和调试的模式


# Remix
有几种solidity的IDE可供选择，以下我会一直用Remix，一种在线并带有debug的IDE，直接访问网址即可：
* http://remix.ethereum.org

# geth
geth是以太坊的客户端，也是以太坊钱包mist的核心。虽然Remix提供了Javascript VM 和 injected web3两种形式可供调试，但是并不能完全满足调试需求。让remix连接本地的geth是一种很好的选择，既可以调试私有链，又可以调试eth的公有链，如果你熟悉geth操作的话，还能减少很多麻烦。
<!-- more -->
## 下载geth
https://geth.ethereum.org/downloads/ <br>
在这个网站上下载1.7.1版本的geth，虽然已经有新的版本更新了，但是有些命令有变动。如果我以后使用1.8版本的话会在这里更新。geth解压以后只有一个可执行文件，名字就叫geth。
## 启动geth
假设我们在私有链中测试合约，我们在geth目录下通过以下命令启动geth以搭建我们的私有链：
```bash
./geth --dev --rpc --rpcaddr 127.0.0.1 --rpcport 8545 --rpcapi "personal,db,eth,net,web3,debug" --gasprice "18000000000" --mine --datadir "./data" --rpccorsdomain "*" console 2>>eth.log
```
简单解释下参数
* --dev 私有链测试模式
* --rpc 开启rpc端口，以供remix能够访问到
* --rpcport rpc端口，默认其实就是
* --rpcapi  开放哪些rpc接口，尤其是debug，remix会需要用到这个
* --mine  自动挖矿
* --gasprice gas的价格，如果不设置测试环境中就为0,那就和共有链不一样了，导致测试结果偏差
* --datadir 区块链资源存放的位置，最好设置一下不然很难找，如果你想删除所有区块链信息和钱包重启私有链，删除这个文件夹就行

终端中启动完geth就放在那就行了，现在我们去remix里面，在右上角的run选项卡里找到Environment，选择web3 provider,如下图

![](/images/solidity1.png)

因为我们默认端口是8545,所以提示输入本地接口地址的时候什么都不用变，点确定就行。如果连接成功会有提示。

## 关于帐号
再进入下一步之前，我们要现在geth中创建一个账户，不然没法完成测试。关于geth的其他操作我在其他文章里会讲，这里只是提一下会用到的命令。打开刚才启动geth的终端，输入以下命令：
### 创建帐号
```javascript
personal.newAccount("password")
```
反复使用创建多个帐号
### 解锁帐号
```javascript
personal.unlockAccount(eth.accounts[0],"password",100000000)
```
最后一个数字写大一点，是解锁的时间，以免remix经常提示帐号没解锁
### 批量解锁帐号
```javascript
var pass = "password"
for(var i=0;i < eth.accounts.length ; i++) {
  personal.unlockAccount(eth.accounts[i],pass,100000000)
}
```
我们经常需要多个帐号调试，所以这里提供一个批量解锁帐号的方法，直接输入到geth终端即可

# 编写并创建一个简单合约
ok，我们可以正式开始创建合约了。我从官网找了一段ERC20合约的示例代码过来，如下：
```solidity
pragma solidity ^0.4.16;

interface tokenRecipient { function receiveApproval(address _from, uint256 _value, address _token, bytes _extraData) public; }

contract TokenERC20 {
    // Public variables of the token
    string public name;
    string public symbol;
    uint8 public decimals = 18;
    // 18 decimals is the strongly suggested default, avoid changing it
    uint256 public totalSupply;

    // This creates an array with all balances
    mapping (address => uint256) public balanceOf;
    mapping (address => mapping (address => uint256)) public allowance;

    // This generates a public event on the blockchain that will notify clients
    event Transfer(address indexed from, address indexed to, uint256 value);

    // This notifies clients about the amount burnt
    event Burn(address indexed from, uint256 value);

    /**
     * Constrctor function
     *
     * Initializes contract with initial supply tokens to the creator of the contract
     */
    function TokenERC20(
        uint256 initialSupply,
        string tokenName,
        string tokenSymbol
    ) public {
        totalSupply = initialSupply * 10 ** uint256(decimals);  // Update total supply with the decimal amount
        balanceOf[msg.sender] = totalSupply;                // Give the creator all initial tokens
        name = tokenName;                                   // Set the name for display purposes
        symbol = tokenSymbol;                               // Set the symbol for display purposes
    }

    /**
     * Internal transfer, only can be called by this contract
     */
    function _transfer(address _from, address _to, uint _value) internal {
        // Prevent transfer to 0x0 address. Use burn() instead
        require(_to != 0x0);
        // Check if the sender has enough
        require(balanceOf[_from] >= _value);
        // Check for overflows
        require(balanceOf[_to] + _value > balanceOf[_to]);
        // Save this for an assertion in the future
        uint previousBalances = balanceOf[_from] + balanceOf[_to];
        // Subtract from the sender
        balanceOf[_from] -= _value;
        // Add the same to the recipient
        balanceOf[_to] += _value;
        Transfer(_from, _to, _value);
        // Asserts are used to use static analysis to find bugs in your code. They should never fail
        assert(balanceOf[_from] + balanceOf[_to] == previousBalances);
    }

    /**
     * Transfer tokens
     *
     * Send `_value` tokens to `_to` from your account
     *
     * @param _to The address of the recipient
     * @param _value the amount to send
     */
    function transfer(address _to, uint256 _value) public {
        _transfer(msg.sender, _to, _value);
    }

    /**
     * Transfer tokens from other address
     *
     * Send `_value` tokens to `_to` in behalf of `_from`
     *
     * @param _from The address of the sender
     * @param _to The address of the recipient
     * @param _value the amount to send
     */
    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        require(_value <= allowance[_from][msg.sender]);     // Check allowance
        allowance[_from][msg.sender] -= _value;
        _transfer(_from, _to, _value);
        return true;
    }

    /**
     * Set allowance for other address
     *
     * Allows `_spender` to spend no more than `_value` tokens in your behalf
     *
     * @param _spender The address authorized to spend
     * @param _value the max amount they can spend
     */
    function approve(address _spender, uint256 _value) public
        returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        return true;
    }

    /**
     * Set allowance for other address and notify
     *
     * Allows `_spender` to spend no more than `_value` tokens in your behalf, and then ping the contract about it
     *
     * @param _spender The address authorized to spend
     * @param _value the max amount they can spend
     * @param _extraData some extra information to send to the approved contract
     */
    function approveAndCall(address _spender, uint256 _value, bytes _extraData)
        public
        returns (bool success) {
        tokenRecipient spender = tokenRecipient(_spender);
        if (approve(_spender, _value)) {
            spender.receiveApproval(msg.sender, _value, this, _extraData);
            return true;
        }
    }

    /**
     * Destroy tokens
     *
     * Remove `_value` tokens from the system irreversibly
     *
     * @param _value the amount of money to burn
     */
    function burn(uint256 _value) public returns (bool success) {
        require(balanceOf[msg.sender] >= _value);   // Check if the sender has enough
        balanceOf[msg.sender] -= _value;            // Subtract from the sender
        totalSupply -= _value;                      // Updates totalSupply
        Burn(msg.sender, _value);
        return true;
    }

    /**
     * Destroy tokens from other account
     *
     * Remove `_value` tokens from the system irreversibly on behalf of `_from`.
     *
     * @param _from the address of the sender
     * @param _value the amount of money to burn
     */
    function burnFrom(address _from, uint256 _value) public returns (bool success) {
        require(balanceOf[_from] >= _value);                // Check if the targeted balance is enough
        require(_value <= allowance[_from][msg.sender]);    // Check allowance
        balanceOf[_from] -= _value;                         // Subtract from the targeted balance
        allowance[_from][msg.sender] -= _value;             // Subtract from the sender's allowance
        totalSupply -= _value;                              // Update totalSupply
        Burn(_from, _value);
        return true;
    }
}
```
好了，我们现在还不需要直到合约里面到底写了什么，我们只要把这段代码复制到Remix中就可以了。然后选择Compile选项卡，点击 start to compile。没有错误就切换到run选项卡，在create中填入ERC20合约的三个参数，分别是发行量、token名字、token简称，像这样：
```javascript
100000,"TinyCalf Token","TCT"
```
在address中复制上面的帐号地址下来，点击create。左下角会提示pending，等挖矿，下一个块生成后，合约就创建成功了。如图

![](/images/solidity2.png)

点击下面的各种函数的按钮就可以完成合约函数的调用

# 调试合约交易
我们点击下面的tototalSupply,symbol,name这些方法，可以直接显示结果，因为这些函数调用只需查看区块链就能实现。而很多方法需要通过向合约发送数据才能实现，比如transfer方法，就是把一个帐号中的token发到另一个帐号，这个过程相当于我要告诉合约，我要转给B一些token，合约来记录这个过程，所以这个方法相当于发给合约的一个交易，将调用什么方法，需要哪些参数告诉了合约。我这里阐述得简单一点，大概懂就行，具体的我会在其他文章里讲。回到正题，transfer需要两个参数，目标地址和数量，我们在transfer的输入框中输入：
```javascript
"0xfc9f5c897eda12d1131ecd7b7db2fa67fb28c24c","1000000000"
```
记得把地址改成你创建的另一个，点击transfer，这时后又会显示pending，正如我所说，这个方法本身是一个交易。等到新块生成，交易确认以后，我们就可以在左边的输出框中点debug来调试这笔交易，如图
![](/images/solidity3.png)
这个操作界面我就不多说了，还是比较简单明了的，能逐步运行函数，检查每个变量的值。

# 简单解释下合约的原理
有一点需要明确，合约不是在区块链中执行的，而是在每个节点本地执行的。区块链上记录的是什么帐号调用了什么方法。本地执行了合约从创建开始到现在，所有区块链上记录的函数调用，以得出一个最终的状态。合约地址本身是一个没有人直到私钥的帐号，可以接收toke和ether。token是合约的一个应用，传递token只是一个合约方法，原理是告诉合约A要转给B多少个token，合约里面记录了每个地址的token数量，和ether的原理不一样。有问题的直接留言，本篇结束。

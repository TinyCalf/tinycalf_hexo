---
title: 以ERC20为例讲解Solidity编程
date: 2018-05-31 15:33:54
categories: Solidity
author: TinyCalf
tags:
- solidity
---

## ERC20示例代码

以下是一段官方提供的ERC20的代码, 省略了实现部分，完整代码请参考我上传的demo
```
pragma solidity ^0.4.16;

contract TokenERC20 {
    uint256 public totalSupply;
    mapping (address => uint256) public balanceOf;
    mapping (address => mapping (address => uint256)) public allowance;
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Burn(address indexed from, uint256 value);

    function TokenERC20(
        uint256 initialSupply,
        string tokenName,
        string tokenSymbol) public

    function _transfer(address _from, address _to, uint _value) internal

    function transfer(address _to, uint256 _value) public

    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)

    function approve(address _spender, uint256 _value) public
        returns (bool success)

    function approveAndCall(address _spender, uint256 _value, bytes _extraData)
        public
        returns (bool success)

    function burn(uint256 _value) public returns (bool success)

    function burnFrom(address _from, uint256 _value) public returns (bool success)
}
```
<!-- more -->

可以从以上的代码中看到ERC20标准的组成元素有这些
* totalSupply   总发行量
* balanceOf     记录所有地址对应token余额的map
* allowance     记录地址提取权限的map(ERC20有允许其他地址提取自己token的功能)
* Transfer      是一个事件，表示产生了本token的交易
* Burn          是一个事件，表示销毁了多少token
* TokenERC20    合约在创建时候的构造函数
* \_transfer    完成一笔交易的内部函数
* transfer      转账给其他地址
* transferFrom  从其他地址取出token,前提是有对应的allowance
* approve       给某个地址提取自己token的权限
* burn          销毁token
* burnFrom      从某个地址中销毁token

后面我会挑一些关键的元素来讲解


## ERC20合约解析

### totalSupply
```
uint256 public totalSupply;
```
#### 变量类型
这里定义了一个变量 `totalSupply`，其中 `uint256` 定义了变量的类型，solidity还包含其他变量，如boolean、address等，详细看 [这里](http://solidity.readthedocs.io/en/v0.4.21/types.html#value-types). `totalSupply`是一个可能很大的整数，所以定义为 `uint256`。 Solidity中没有浮点型的数。

#### 可见性
`totalSupply` 中有public关键字，表示该参数或者函数的可见性，可见性包含 public/private/external/interal 四个关键字，而public的用法和正常语言中的public不太一样。详细可以看[这里](https://solidity.readthedocs.io/en/v0.4.21/assembly.html?#solidity-assembly)
。

总结起来说，按权限由小到大排序应该是这样 private < internal < external < public
* private 只能在合约内部被调用
* internal 能在合约内部被调用,也能在继承合约中被调用
* external 不能在合约内部或继承合约中被调用，除非用this关键字；可以通过合约嵌套或者外部api调用
* public 哪里都可以调用

`totalSupply`功能是让外界知道合约的总发行量，所以使用`public`关键字，并会默认生成getter，外部可通过getter获取该变量值

### balanceOf
```
mapping (address => uint256) public balanceOf;
```
这里有一种新的变量类型，`mapping`，可以简单的理解为存放键值对的容器，每个地址对应一个token的余额数量，所以以 address -> uint256 来建立这个容器。这里可以通过 `balanceOf([某个地址])`
来获取地址上的余额

### TokenERC20
```
function TokenERC20(
        uint256 initialSupply,
        string tokenName,
        string tokenSymbol
    ) public {
        totalSupply = initialSupply * 10  uint256(decimals);  // Update total supply with the decimal amount
        balanceOf[msg.sender] = totalSupply;                // Give the creator all initial tokens
        name = tokenName;                                   // Set the name for display purposes
        symbol = tokenSymbol;                               // Set the symbol for display purposes
    }

```
和合约名字一样的函数是构造函数，只有在创建合约的时候会被调用，传递的三个值分别是 发行量、token名称、token简称.

* `balanceOf[msg.sender] = totalSupply` 这句的功能是给合约发起者所有初始化的token,`msg.sender`是一个特殊变量，表示该函数调用的发起者，类型是`address`，还有其他特殊变量，详细可以看[这里](https://solidity.readthedocs.io/en/v0.4.21/units-and-global-variables.html#special-variables-and-functions)

###  \_transfer
```
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
```
* 该函数类型为internal，就像上文说的，这个函数只能在合约内部，或者继承的合约中调用。这个方法处理了token从一个地址转移到另一个地址的简单逻辑
#### 错误处理

```
require(_to != 0x0);
// Check if the sender has enough
require(balanceOf[_from] >= _value);
// Check for overflows
require(balanceOf[_to] + _value > balanceOf[_to]);

...

// Asserts are used to use static analysis to find bugs in your code. They should never fail
assert(balanceOf[_from] + balanceOf[_to] == previousBalances);
```
这个方法用到了require，错误处理方式还有assert和Revert,详见[这里](https://solidity.readthedocs.io/en/v0.4.24/control-structures.html#error-handling-assert-require-revert-and-exceptions)
* require验证括号内的表达式为false时，方法就会退出，require下方的代码不会执行也不会消耗gas（智能合约中的代码每走一步都需要相应的gas，没用完的gas会退还）
* assert中一般是不太可能发生的错误，但是一旦发生错误，整个方法调用就会回退，防止严重的漏洞或者错误发生。比如这里用来验证交易后结果是否正确。

#### 事件
```
Transfer(_from, _to, _value);
```
`Transfer`是最早定义的一个事件（Event）。Event用来向外部抛出重要的事件，比如这里的转账，这样外部就可以通过`Transfer`事件来获取所有token的转账信息。以太坊的钱包就是通过Transfer这个事件来确定一笔转账的。

## 总结
ERC20的标准合约目前就用到了这些功能：变量类型、函数、可见性、错误处理和事件。虽然不是Solidity的全部，但是足以用来应对一些简单的功能。更多信息我会在以后讲解其他合约的时候再详细说

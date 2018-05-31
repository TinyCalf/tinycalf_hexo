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


## ERC20示例代码

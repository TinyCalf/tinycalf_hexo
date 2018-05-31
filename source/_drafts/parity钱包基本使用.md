---
title: parity钱包基本使用
tags:
---

* parity github项目地址 https://github.com/paritytech/parity
https://wiki.parity.io/

安装rustup
```bash
curl https://sh.rustup.rs -sSf | sh
```

sudo apt-get install openssl libssl-dev libudev-dev

```bash
vi ～/.bashrc
```
在里面最有一行加入cargo路径：

```bash
export PATH="$PATH:$HOME/.cargo/bin"
```
用source应用
```bash
source ～/.bashrc
```
编译parity
```bash
# download Parity code
$ git clone https://github.com/paritytech/parity
$ cd parity

# build in release mode
$ cargo build --release
```


sudo dpkg -i parity_1.10.1_ubuntu_amd64.deb

```
sudo apt-get install gcc-4.9 g++-4.9

```

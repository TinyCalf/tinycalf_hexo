---
title: readline-ui和commander打造node可执行脚本
date: 2018-04-12 16:45:33
tags:
- nodejs
categories: nodejs
author: TinyCalf
---
> commander可以让你的node脚本像标准可执行文件一样拥有可选参数和帮助界面，readline可以给node脚本增加交互功能

## npm项目连接
* commander: https://www.npmjs.com/package/commander
* readline-ui: https://www.npmjs.com/package/readline-ui

<!-- more -->

## commander
### 安装
```bash
npm install commander --save
```
### 用例
我就举个最简单的例子，其他具体功能看源项目。 假设我们有一个更改名字的脚本，叫changename
```bash
#!/usr/bin/env node
var program = require('commander');
program
  .version('0.0.1')
  .usage(`-n TinyCalf`)
  .option('-n, --name [name]', 'your name')
  .parse(process.argv);
console.log(program.name)
```
* `#!/usr/bin/env node` 用来让系统用node来执行这个脚本，这样你只需要用 `./changename`来启动脚本，而不需要使用 `node changename`
* `usage` 告诉用户这个脚本该怎么用
* `option` 中规定参数名和描述
* 通过 program[参数名]就能获取用户输入的参数值

然后我们保存退出，给这个changename增加可执行权限
```bash
sudo chmod +x changename
```
然后就可以执行changename了，我们先运行帮助
```bash
./changename -h

  Usage: changename -n TinyCalf

  Options:

    -V, --version      output the version number
    -n, --name [name]  your name
    -h, --help         output usage information
```
刚我们设置的用法久都显示出来了，我们再运行带参数的：
```
./changename -n tinycalf
tinycalf
```
成功获取到了name的值

## readline-ui
### 安装
```bash
npm install --save readline-ui
```
### 用例
还是刚才的例子，我们给changename加一个用户确认的交互，需要用户输入 `Yes！`才能记录操作：
```bash
#!/usr/bin/env node
var program = require('commander');

var UI = require('readline-ui');
var ui = new UI();

program
  .version('0.0.1')
  .usage(`-n TinyCalf`)
  .option('-n, --name [name]', 'your name')
  .parse(process.argv);


var prompt = `Are you sure to change name to ${program.name}? (Yes!) `;
ui.render(prompt);
ui.on('keypress', function() {
  ui.render(prompt + ui.rl.line);
});
ui.on('line', function(answer) {
  ui.render(prompt + answer);
  ui.end();
  ui.rl.pause();
  ui.close();
  if(answer != "Yes!") process.exit()
  console.log(`Your name has change to ${program.name}`)
});
```

*  ui.render 就是发出一个交互，内容为一个字符串
*  对keypress侦听，将每一个键入的字符连接到交互上，让用户看到自己的输出
*  line事件就是对回车键的侦听，我们就当作是用户输入完成的信号，然后就可以判断用户输入的是不是 Yes！

输出大概是这样：
```bash
./changename -n tinycalf
Are you sure to change name to tinycalf? (Yes!) Yes!
Your name has changed to tinycalf
```

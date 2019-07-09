---
title: 监控排除nodejs内存泄漏
date: 2019-10-17 17:23:40
tags:
- nodejs
categories: nodejs
author: TinyCalf
---
> 项目比较庞大的时候，定位内存泄漏会比较难，而且一般已经出现OOM才暴露出内存问题；即使发现并确认内存泄漏，也比较难很快查找到原因；所以本篇讲的是通过heapdump和memwatch两个模块监控和定位内存泄漏。

<!-- more -->

### 一个内存泄漏的例子
下面是一个比较典型的闭包引起的内存泄漏：
```javascript
/* index.js */
var theThing = null;
var replaceThing = function() {
    let 泄漏变量 = theThing;
    let unused = function() {
        if (泄漏变量)
            console.log("hi");
    };
    // 不断修改引用
    theThing = {
        longStr: new Array(1000000).join('*'),
        someMethod: function() {
            console.log('a');
        }
    };
    // 每次输出的值会越来越大
    console.log(process.memoryUsage().heapUsed);
};
setInterval(replaceThing, 100);
```
这段代码中的闭包导致泄漏变量循环引用而不释放，每次运行replaceThing内存占用都会回来越大，最后导致OOM。关于这个例子的讨论可以看[这里](https://github.com/ElemeFE/node-interview/issues/7)；帮助理解上面这个例子的重点是这句话：
>在V8当前版本中，闭包对象是当前作用域中的所有内部函数作用域共享的，并且这个当前作用域的闭包对象中除了包含一条指向上一层作用域闭包对象的引用外，其余的存储的变量引用一定是当前作用域中的所有内部函数作用域中使用到的变量

为了理解一下闭包，还可以看[这里](https://segmentfault.com/a/1190000006875662)；
比较帮助理解的是这两句话：
>MDN 上面这么说：闭包是一种特殊的对象。它由两部分构成：函数，以及创建该函数的环境。环境由闭包创建时在作用域中的任何局部变量组成。

>在javascript中，如果一个对象不再被引用，那么这个对象就会被垃圾回收机制回收；如果两个对象互相引用，而不再被第3者所引用，那么这两个互相引用的对象也会被回收。

下文的例子，都会以这段代码来做测试。

### heapdump
heapdump的作用是保存当前状态的堆栈快照，使用方式非常简单。
安装：
```bash
npm install heapdump
```
在程序入口文件的开头引用：
```javascript
var heapdump = require('heapdump');
```
然后启动我们刚才写的那个内存泄漏的程序，向该进程发送信号，即可在当前目录下生成heapsnapshot文件：
```bash
kill -USR2 <pid>
```
这个文件比较大，我们可以加载到谷歌浏览器的开发者工具中进行分析；打开开发者工具，点击上方Memory标签页，在左侧加载刚才生成的heapsnapshot，如下图：
![图](/images/heapsnapshot1.png)
可以看到closure和string占的比重非常高，我们打开就发现了有很多重复的String内容；再看下面这张图，closure中重复的内容也非常多：
![图](/images/heapsnapshot2.png)
这个时候，看起来我们好像挺容易发现问题的原因的，闭包很多，变量重复，根据上图的内容其实也很容易找到出问题的代码。
### node-memwatch
node-memwatch作用有三：
* 反应垃圾回收的基本状态
* 监控是否出现内存泄漏
* 打印堆内存差异

安装使用很简单，安装：
```bash
npm install node-memwatch
```
我用的node版本是10.15.3，目前发现memwatch和memwatch-next都不能成功安装，只有node-memwatch可以。
然后还是在入口文件里引用就行：
```javascript
var memwatch = require('node-memwatch');
```
#### stats事件
stats事件用来反映垃圾回收的基本状态：
```javascript
memwatch.on('stats', function(stats) {
    console.log('stats:');
    console.log(stats);
});
```
输出如下：
```javascript 
{ num_full_gc: 15, //第几次全堆垃圾回收
  num_inc_gc: 15, //第几次增量垃圾回收
  heap_compactions: 15, //第几次对老生代进行整理
  usage_trend: 3.6, //使用趋势
  estimated_base: 110243248, //预估基数
  current_base: 158730872, //当前基数
  min: 55708632, //最小
  max: 158730872 } //最大
```
#### leak事件
经过5次垃圾回收以后，内存仍然没有被释放的话，会触发leak事件，表示可能存在内存泄漏了。
代码
```javascript 
memwatch.on('leak', function(info) {
    console.log('leak:');
    console.log(info);
});
```
输出如下：
```javascript
leak:
{ growth: 27263856,
  reason:
   'heap growth over 5 consecutive GCs (4s) - -2147483648 bytes/hr' }
```
这个leak事件其实就是警报作用，具体如何定位问题，还需要下面的第三个功能
#### 输出堆内存差异
在n个事件循环中，如果没有内存泄漏，每个对象的差异理论上应该差不多，memwatch的这个功能，就是依照这个理论。我们在发现leak事件以后，比较一段时间里的堆内存快照，就能比较容易确定造成内存泄漏的内容。所以我们在leak时间中增加这些逻辑：
```javascript
memwatch.on('leak', function(info) {
    console.log('leak:');
    console.log(info);
    var hd = new memwatch.HeapDiff(); 
    setTimeout(function(){
        var diff = hd.end();
        console.log(JSON.stringify(diff));
    }, 500);
});
```
代码运行一段时间以后，我们得到：
```javascript
{ before: { nodes: 34281, size_bytes: 121738247, size: '116.1 mb' },
  after: { nodes: 34328, size_bytes: 126739327, size: '120.87 mb' },
  change:
   { size_bytes: 5001080,
     size: '4.77 mb',
     freed_nodes: 115,
     allocated_nodes: 162,
     details:
     [ { what: 'Array',
       size_bytes: 64,
       size: '64 bytes',
       '+': 13,
       '-': 12 },
     { what: 'Closure',
       size_bytes: 3072,
       size: '3 kb',
       '+': 48,
       '-': 0 },
     { what: 'Native',
       size_bytes: 656,
       size: '656 bytes',
       '+': 59,
       '-': 58 },
     { what: 'Number',
       size_bytes: 16,
       size: '16 bytes',
       '+': 1,
       '-': 0 },
     { what: 'Object',
       size_bytes: 1768,
       size: '1.73 kb',
       '+': 47,
       '-': 2 },
     { what: 'String',
       size_bytes: 47000904,
       size: '44.82 mb',
       '+': 48,
       '-': 4 },
     { what: 'TickObject',
       size_bytes: -48,
       size: '-48 bytes',
       '+': 0,
       '-': 2 },
     { what: 'Timeout',
       size_bytes: 176,
       size: '176 bytes',
       '+': 1,
       '-': 0 },
     { what: 'Timer', size_bytes: 32, size: '32 bytes', '+': 1, '-': 0 },
     { what: 'TimersList',
       size_bytes: 128,
       size: '128 bytes',
       '+': 1,
       '-': 0 },
     { what: 'system / Context',
       size_bytes: 2632,
       size: '2.57 kb',
       '+': 47,
       '-': 0 }
      ] } 
```
列出了发现内存泄漏以后，每个对象的大小，增加的数量，和减少的数量（+和-）；如果某个对象增加的数量比减少的数量大很多，那么很有可能是泄漏的变量；还可以通过增加的大小来判断某变量造成了多少内存的使用；这两点对于我们判断内存泄漏，会有很大帮助。比如上面给出的结果中，我们就可以看到Closure和String数量和容量都增加了很多，但是没有减少。

### 目前的实践
heapdump和memwatch各有各的好处和不足，结合起来用其实效果还不错。我们可以通过memwatch对内存泄漏进行报警，通过堆栈比较找出可能引起内存泄漏的对象；但是memwatch无法直接定位到代码，heapdump因为无法比较堆栈差异，所以在内容庞大的情况下很难定位问题；那么可以结合使用heapdump和memwatch才找到具体出问题的地方。

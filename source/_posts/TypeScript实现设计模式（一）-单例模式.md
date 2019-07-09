---
title: TypeScript实现设计模式（一）-- 单例模式
date: 2019-07-06 16:51:37
tags:
- 设计模式
- TypeScript
- nodejs
categories: 设计模式
author: TinyCalf
---
> 最近在阅读《设计模式之禅》，一方面想实践实践，另一方面ts和java的实现有很多不同之处，所以引用一些书中的例子用ts的方法实现；具体设计模式详解我就不在这写啦，毕竟书里和掘金上已经有很多；可能不会所有模式都写，看情况吧；今天先写第一个：单例模式（Singleton）。

<!-- more -->
### 饱汉模式
即申明时就创建单例，书中没提饱汉饿汉两个名次，但是两种实现都写了。下面是饱汉模式的实现：
```typescript
/**
 * @file Emperor.js
 */
class Emperor {
    private static emperor: Emperor = new Emperor("嬴政");
    private name: string;

    private constructor(name: string) {
        this.name = name;
    }

    public static getInstance(): Emperor {
        return Emperor.emperor;
    }

    public say(): void {
        console.log(`我是皇帝${this.name}`);
    }
}
```

有一个static变量emperor和static方法getInstance，是实现单例模式的关键；
ts的static关键字和java中的功能一样，我们看看编译出来的js代码：
```javascript
var Emperor = /** @class */ (function () {
    function Emperor(name) {
        this.name = name;
    }
    Emperor.getInstance = function () {
        return Emperor.emperor;
    };
    Emperor.prototype.say = function () {
        console.log("\u6211\u662F\u7687\u5E1D" + this.name);
    };
    Emperor.emperor = new Emperor("嬴政");
    return Emperor;
}());
```
所以实际都是在Emperor类下面直接定义了emperor和getInstance，而不是在prototype下。因此调用他们和java也略有所区别，在getInstance中，使用了
```typescript
    return Emperor.emperor;
```
而不通过this或者直接使用emperor来访问；获取Emperor实例也一样,不用new关键字，我们这么写：
```typescript
let emperor = Emperor.getInstance();
emperor.say(); //stdout: 我是皇帝嬴政
```

### 饿汉模式
即在第一次使用的时候创建实例；nodejs是单线程的所以不需要考虑线程安全，所以这里简单多了：
```typescript
class Emperor {
    private static emperor: Emperor = null;
    private name: string;

    private constructor(name: string) {
        this.name = name;
    }

    public static getInstance(): Emperor {
        if (Emperor.emperor == null) {
            Emperor.emperor = new Emperor("嬴政");
        }
        return Emperor.emperor;
    }

    public say(): void {
        console.log(`我是皇帝${this.name}`);
    }
}
```
ts实现单例模式就介绍到这里。

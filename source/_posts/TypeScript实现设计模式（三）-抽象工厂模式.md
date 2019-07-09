---
title: TypeScript实现设计模式（三）-- 抽象工厂模式
date: 2019-07-10 21:00:15
tags:
- 设计模式
- TypeScript
- nodejs
categories: 设计模式
author: TinyCalf
---
> 抽象工厂模式（Abstruct Factory Pattern）：为创建一组相关或相互依赖的对象提供一个接口，而无须指定他们的具体类。笔者简单推演了一下工厂方法模式到抽象工厂模式的转变过程。

<!-- more -->

抽象工程模式其实就是工厂方法模式的扩展；这回我们举一个具体点的例子：3G时代的时候，苹果和华为都找富士康代工生产3G手机，我们先用工厂方法模式实现一下，和上一篇差不多：

```TypeScript
abstract class MobilePhone3G {
    public abstract call(number: string): void;
    public abstract sendSMS(number: string, ctx: string): void;
}

class Iphone4S extends MobilePhone3G {
    public call(number: string): void {
        console.log(`calling ${number}...`);
    }
    public sendSMS(number: string, ctx: string): void {
        console.log(`sending ${ctx} to ${number}`);
    }
}

class HuaweiMate extends MobilePhone3G {
    public call(number: string): void {
        console.log(`calling ${number}...`);
    }
    public sendSMS(number: string, ctx: string): void {
        console.log(`sending ${ctx} to ${number}`);
    }
}

abstract class Factory {
    public abstract createMobilePhone3G<T extends MobilePhone3G>(C: new () => T): T;
}

class FoxConnFactory extends Factory {
    public createMobilePhone3G<T extends MobilePhone3G>(C: new () => T): T {
        let phone: T = null;
        try {
            phone = new C();
        } catch (e) {
            console.error("produce phone failed！");
        }
        return phone;
    }
}

let factory = new FoxConnFactory();
let phone = factory.createMobilePhone3G(Iphone4S);
phone.call("15082888128"); //stdout: calling 15082888128...
```
ok，很简单，富士康用一家工厂同时生产两个品牌的手机。但是这个方法遇到了第一个问题，那就是华为和苹果有各自的核心科技，组装方法并不一样，在一家工厂生产难免会碰到各种麻烦，为了体现这里麻烦，我们让苹果和华为手机的构造方法不一样一点，各自组装了自己的核心科技：
```TypeScript
class Iphone4S extends MobilePhone3G {

    appleSecretModule: string;

    public constructor(appleSecretModule: string) {
        super();
        this.appleSecretModule = appleSecretModule;
    }
    public call(number: string): void {
        console.log(`calling ${number}...`);
    }
    public sendSMS(number: string, ctx: string): void {
        console.log(`sending ${ctx} to ${number}`);
    }
}

class HuaweiMate extends MobilePhone3G {

    huaweiSecretModule: number;

    public constructor(huaweiSecretModule: number) {
        super();
        this.huaweiSecretModule = huaweiSecretModule;
    }

    public call(number: string): void {
        console.log(`calling ${number}...`);
    }
    public sendSMS(number: string, ctx: string): void {
        console.log(`sending ${ctx} to ${number}`);
    }
}
```
因为构造方法不一样了，用一个工厂生产显然不太合理，于是我们继承同一个工厂开设两家工厂，分别生产苹果和华为手机：
```TypeScript
class AppleFactory extends Factory {
    public createMobilePhone3G<T extends MobilePhone3G>(C: new (appleSecretModule: string) => T): T {
        let phone: T = null;
        let appleSecretModule: string = "UDIIKS@！##¥";
        try {
            phone = new C(appleSecretModule);
        } catch (e) {
            console.error("produce phone failed！");
        }
        return phone;
    }
}

class HuaweiFactory extends Factory {
    public createMobilePhone3G<T extends MobilePhone3G>(C: new (huaweiSecretModule: number) => T): T {
        let phone: T = null;
        let huaweiSecretModule: number = 100192;
        try {
            phone = new C(huaweiSecretModule);
        } catch (e) {
            console.error("produce phone failed！");
        }
        return phone;
    }
}
```
然后生产一台iphone4S：
```TypeScript
let appleFactory = new AppleFactory();
let iphone4S = appleFactory.createMobilePhone3G(Iphone4S);
iphone4S.call("15082888128");//stdout: calling 15082888128...
```
经过现在的调整富士康的生产线的扩展性已经大大提高了，现在要是其他手机厂商需要代工，直接新建一个新的工厂即可，管你有什么核心科技都无所谓了。

又过了几年，移动网络进入了4G时代，各大厂商纷纷开发了4G手机,这个时候我们需要创建一下4G手机的虚类：
```TypeScript
abstract class MobilePhone4G {
    public abstract call(number: string): void;
    public abstract sendSMS(number: string, ctx: string): void;
}
```
然后华为和苹果设计出了4G手机并且都保留了他们的核心科技：
```TypeScript
class Iphone6Plus extends MobilePhone4G {

    appleSecretModule: string;

    public constructor(appleSecretModule: string) {
        super();
        this.appleSecretModule = appleSecretModule;
    }
    public call(number: string): void {
        console.log(`calling ${number}...`);
    }
    public sendSMS(number: string, ctx: string): void {
        console.log(`sending ${ctx} to ${number}`);
    }
}

class HuaweiP9 extends MobilePhone4G {

    huaweiSecretModule: number;

    public constructor(huaweiSecretModule: number) {
        super();
        this.huaweiSecretModule = huaweiSecretModule;
    }

    public call(number: string): void {
        console.log(`calling ${number}...`);
    }
    public sendSMS(number: string, ctx: string): void {
        console.log(`sending ${ctx} to ${number}`);
    }
}
```
因为核心科技基本相同，生产苹果的工厂没必要分成两家工厂，其他的也一样，所以，我们在同一家工厂里同时生产3G和4G的手机：
```TypeScript
abstract class Factory {
    public abstract createMobilePhone3G<T extends MobilePhone3G>(C: new () => T): T;
    public abstract createMobilePhone4G<T extends MobilePhone3G>(C: new () => T): T;
}
```
然后苹果手机工厂就变成这样：
```TypeScript
class AppleFactory extends Factory {
    appleSecretModule: string = "UDIIKS@！##¥";
    public createMobilePhone3G<T extends MobilePhone3G>(C: new (appleSecretModule: string) => T): T {
        let phone: T = null;
        try {
            phone = new C(this.appleSecretModule);
        } catch (e) {
            console.error("produce phone failed！");
        }
        return phone;
    }

    public createMobilePhone4G<T extends MobilePhone4G>(C: new (appleSecretModule: string) => T): T {
        let phone: T = null;
        try {
            phone = new C(this.appleSecretModule);
        } catch (e) {
            console.error("produce phone failed！");
        }
        return phone;
    }
}
```
目前我们不管是横向扩展还是纵向扩展，都没有问题了，简直完美，5G6G来了也不怕。经过上面几步，我们其实从最开始的工厂方法模式，过渡到了抽象工厂模式。当然最后还是要提一点，抽象工厂模式很庞大，看看我们上面的代码就知道了，并不是所有情况我们都需要一个抽象工厂，如果一个模块很显然需要在多个维度上扩展，那才可能考虑用这个模式。例如一个系统需要在Windows/Linux/Android上同时开发，我们并不需要开发多份代码，只需要用不同的工厂方法处理与不同系统的交互。

抽象方法模式就总结到这里，有什么写的不好的地方请多多指正。

这个代码高亮我一定要弄好它！！！·

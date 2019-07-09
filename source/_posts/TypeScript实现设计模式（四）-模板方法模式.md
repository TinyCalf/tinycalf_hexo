---
title: TypeScript实现设计模式（四）-- 模板方法模式
date: 2019-07-11 15:07:47
tags:
- 设计模式
- TypeScript
- nodejs
categories: 设计模式
author: TinyCalf
---
> 模板方法模式（Template Method Pattern）: 定义一个操作中算法的框架，而将一些步骤延迟到子类中。使得子类可以不改变一个算法的结构即可重定义算法的某些特定步骤。

<!-- more -->

这回我们还是以手机为例，但是具体到如何生产一部手机，所以我们定义生产iphone的流水线的抽象;
一部iphone需要安装主板、安装核心科技、安装摄像头、安装外壳，然后把所有过程串联起来，但是每款iphone安装的方式又不尽相同，所以我们抽象了这些方法，让具体的生产线去实现如何安装：

```TypeScript
abstract class IPhonePipeline {
    protected abstract installMainboard(): void;
    protected abstract installCoreTech(): void;
    protected abstract installCamera(): void;
    protected abstract installShell(): void;
    public abstract produceIPhone(): void;
}
```
好了，我们现在分别开一条iphone4s的生产线和iphone6plus的生产线：
```TypeScript
class IPhone4SPipeline extends IPhonePipeline {
    protected installMainboard(): void {
        console.log('iphone4S installing Mainboard...');
    }
    protected installCoreTech(): void {
        console.log('iphone4S installing CoreTech...');
    }
    protected installCamera(): void {
        console.log('iphone4S installing Camera...');
    }
    protected installShell(): void {
        console.log('iphone4S installing installShell...');
    }

    public produceIPhone(): void {
        this.installMainboard();
        this.installCoreTech();
        this.installCamera();
        this.installShell();
        console.log('completed install iphone 4s');
    }
}

class IPhone6plusPipeline extends IPhonePipeline {
    protected installMainboard(): void {
        console.log('iPhone6plus installing Mainboard...');
    }
    protected installCoreTech(): void {
        console.log('iPhone6plus installing CoreTech...');
    }
    protected installCamera(): void {
        console.log('iPhone6plus installing Camera...');
    }
    protected installShell(): void {
        console.log('iPhone6plus installing installShell...');
    }

    public produceIPhone(): void {
        this.installMainboard();
        this.installCoreTech();
        this.installCamera();
        this.installShell();
        console.log('completed install iPhone6plus');
    }
}
```
然后生产一下试试：
```TypeScript
let pipeline = new IPhone4SPipeline();
pipeline.produceIPhone();
//stdout:
//iphone4S installing Mainboard...
//iphone4S installing CoreTech...
//iphone4S installing Camera...
//iphone4S installing installShell...
//completed install iphone 4s
```
每多一部iphone就可以多一条生产线，写法同上。写到这里你们发现什么没？安装各种模块的时候各不相同，可以理解，但是最终组装的时候方法都是完全一样的呀。所以我们找到了所有生产线的共性，就是produceIPhone，于是模板方法模式就派上用场了，相同的方法我们可以总结到模板里，而不需要每个生产线分别去实现，减少了很多麻烦不说，还保证了各个生产线不会生产错。那我们先修改模板类，也就是我们的生产线虚类：
```TypeScript
abstract class IPhonePipeline {
    protected abstract installMainboard(): void;
    protected abstract installCoreTech(): void;
    protected abstract installCamera(): void;
    protected abstract installShell(): void;
    public produceIPhone(): void {
        this.installMainboard();
        this.installCoreTech();
        this.installCamera();
        this.installShell();
        console.log('completed install iphone');
    }
}
```
**一般情况下，模板方法模式要求模板方法增加final关键字，以防被子类覆写，但是TypeScript目前还没有final关键字，和final类似的关键字readonly也只能在属性中使用，所以目前，只能自己注意点别覆盖模板方法。** 有了模板方法以后，我们的生产线就可以简单点了：
```TypeScript
class IPhone4SPipeline extends IPhonePipeline {
    protected installMainboard(): void {
        console.log('iphone4S installing Mainboard...');
    }
    protected installCoreTech(): void {
        console.log('iphone4S installing CoreTech...');
    }
    protected installCamera(): void {
        console.log('iphone4S installing Camera...');
    }
    protected installShell(): void {
        console.log('iphone4S installing installShell...');
    }
}

class IPhone6plusPipeline extends IPhonePipeline {
    protected installMainboard(): void {
        console.log('iPhone6plus installing Mainboard...');
    }
    protected installCoreTech(): void {
        console.log('iPhone6plus installing CoreTech...');
    }
    protected installCamera(): void {
        console.log('iPhone6plus installing Camera...');
    }
    protected installShell(): void {
        console.log('iPhone6plus installing installShell...');
    }
}

let pipeline = new IPhone4SPipeline();
pipeline.produceIPhone();
//stdout:
//iphone4S installing Mainboard...
//iphone4S installing CoreTech...
//iphone4S installing Camera...
//iphone4S installing installShell...
//completed install iphone
```
这就是模板方法模式，是不是太简单了？再简单也是模式嘛。现在再提出一个问题，每个手机外壳都要上色，上色方法其实都一样，只不过需要上的颜色不同而已，这个时候就用到了模板方法模式的扩展。要申明的是子类重写produceIPhone是必须被禁止的，虽然可以这么做，那也只是因为ts没有final关键字而已。在这种情况下我们想让子类影响父类的方法怎么办？那我们要请出进场能见到的 **钩子方法**:
```Typescript
enum Color {
    White = 'white',
    Black = 'black',
    Sliver = 'sliver'
};

abstract class IPhonePipeline {
    protected abstract installMainboard(): void;
    protected abstract installCoreTech(): void;
    protected abstract installCamera(): void;
    protected abstract installShell(): void;
    protected getPaintColor(): Color {
        return Color.Black;
    }
    public produceIPhone(): void {
        this.installMainboard();
        this.installCoreTech();
        this.installCamera();
        this.installShell();
        console.log(`painting color ${this.getPaintColor()}`);
        console.log('completed install iphone');
    }
}

class IPhone4SPipeline extends IPhonePipeline {
    private color: Color = Color.White;
    protected installMainboard(): void {
        console.log('iphone4S installing Mainboard...');
    }
    protected installCoreTech(): void {
        console.log('iphone4S installing CoreTech...');
    }
    protected installCamera(): void {
        console.log('iphone4S installing Camera...');
    }
    protected installShell(): void {
        console.log('iphone4S installing installShell...');
    }

    public getPaintColor(): Color {
        return this.color;
    }

    public setColor(color: Color): boolean {
        this.color = color;
        return true;
    }
}
```
在虚类IPhonePipeline中我们看到定义了getPaintColor方法，固定返回黑色，这就是我们的钩子方法，子类IPhone4SPipeline通过覆写getPaintColor方法影响了父类produceIPhone方法的执行，有没有似曾相识？很多框架比如游戏服务器框架pomelo的通用组件类Component就是模板方法，实现一个Component就能让该组件被pomelo加载，并且提供的before/after等钩子函数让我们能控制被加载过程中执行的方法，而且我们可以无限扩展各种组件。

模板方法模式
优势：
* 封装不变部分，扩展可变部分
* 提取公共部分代码，便于维护
* 行为由父类控制，子类实现

劣势：
* 阅读有点难度

使用场景：
* 有能提取的公共方法
* 重要复杂的算法
* 需要大量扩展的方法类

模板方法模式就写到这里。

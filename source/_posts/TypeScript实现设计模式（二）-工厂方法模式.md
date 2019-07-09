---
title: TypeScript实现设计模式（二）-- 工厂方法模式
date: 2019-07-09 20:27:14
tags:
- 设计模式
- TypeScript
- nodejs
categories: 设计模式
author: TinyCalf
---
> 工厂方法模式（Factory Method Pattern）:定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法是使用频率最高的设计模式之一；根据具体情况有多个变种；以下是TypeScript的实现。

<!-- more -->
## 标准工厂方法模式
首先我们需要一个产品的抽象层，用来约定所有产品需要实现哪些方法；interface和abstract class都可以对产品做抽象，选任何一种都可以，根据具体情况而定；这里举例用interface作为产品的抽象：
```TypeScript
interface Product {
    m1(): void;
    m2(): void;
}
```

然后我们来实现两种产品：
```TypeScript
class ProductA implements Product {
    public m1(): void {
        console.log('ProductA is calling func m1');
    }

    public m2(): void {
        console.log('ProductA is calling func m2');
    }
}


class ProductB implements Product {
    public m1(): void {
        console.log('ProductB is calling func m1');
    }

    public m2(): void {
        console.log('ProductB is calling func m2');
    }
}
```
还需要一个工厂的抽象层，因为可能要建很多工厂；当然也有只需要一个工厂的场景，后面也会提到；实现如下：
```TypeScript
abstract class AbstractFactory {
    public abstract createProduct<T extends Product>(C: new () => T): T;
}
```
这里实现的是一个标准做法，可以传入要生产的产品的class来获得该产品；`<T extends Product>`定义了模板T为Product的实现；输入参数为Class而非实例，所以C的类型为`new () => T`，C的类型还有另一种写法，如下：
```TypeScript
abstract class AbstractFactory {
    public abstract createProduct<T extends Product>(C: {new(): T; }): T;
}
```
最后实现一下工厂：
```TypeScript
class Factory extends AbstractFactory {
    public createProduct<T extends Product>(C: new () => T): T {
        let product: T = null;
        try {
            product = new C();
        } catch (e) {
            console.error("create product failed！");
        }
        return product;
    }
}
```
生产一个产品试试：
```TypeScript
let factory = new Factory();
let product = factory.createProduct(ProductA);
product.m1(); //ProductA is calling func m1
```
书里介绍过的的我就不多说啦，简单总结一下。我认为工厂方法模式主要提供了两层约束，第一层是约束了产品的实现，通过定义interface或者abstract class保证了产品的“质量”，即满足所需的所有要求；第二层约束是工厂，保证生产出来的是某一类产品，而不是其他什么类的什么产品，想象一下如果不用模式我们一定是用switch来提供各种产品，那switch中返回的是什么是不是完全不可控呢。

## 简单工厂模式
正如刚才说的，有的时候我们真的不需要一个抽象工厂类，因为在可见的未来都不太可能需要多个工厂，那么这个时候我们也不需要强行增加复杂度，我们只需要一个简单工厂，并且提供一个*静态工厂方法*即可，实现如下：
```TypeScript
class SimpleFactory {
    public static createProduct<T extends Product>(C: new () => T): T {
        let product: T = null;
        try {
            product = new C();
        } catch (e) {
            console.error("create product failed！");
        }
        return product;
    }
}
```
生产产品也略有区别，调用的是静态方法：
```TypeScript
let product = SimpleFactory.createProduct(ProductA);
product.m1();//ProductA is calling func m1
```
## 延迟初始化
延迟初始化（Lazy Initialization），就是保留创建的对象，以备再次使用；我举个例子，比如我们开发一个聊天系统，聊天系统中有各种频道，频道被创建时是需要被保留的，并且有可能要控制频道数量，这时候用延迟初始化的方式就很好了，来看我的代码实现：
```TypeScript
abstract class Channel {
    public abstract add(user: string): boolean;
    public abstract kick(user: string): boolean;
}

class GroupChannel extends Channel {
    public add(user: string): boolean {
        return true;
    }
    public kick(user: string): boolean {
        return true;
    }
}

class ChannelService {
    private static readonly channelMap: Map<string, Channel> = new Map();

    public static createChannel(name: string): Channel {
        let channel: Channel = null;
        if (this.channelMap.has(name)) {
            channel = this.channelMap.get(name);
        } else {
            channel = new GroupChannel();
            this.channelMap.set(name, channel);
        }
        return channel;
    }
}
```
代码其实很好理解，ChannelService中用一个Map保存所有channel，这里用到了一个readonly关键字，它和java中的final是等效的，表示被初始化后不可变；除了一个静态的Map我们还实现了静态方法createChannel，如果能找到对应的channel则返回，如果没找到，则创建再返回，这样新的用户就能进入以前创建过的频道。

工厂方法模式几乎没什么缺点，代码结构清晰，可扩展性强，并且使用非常频繁；但是要活学活用，设计模式提供的只是参考，具体如何运用还是得和实际情况相结合。

这个代码高亮好像对ts支持的不太好，我想想办法换一个╮(╯ ╰)╭

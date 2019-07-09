---
title: TypeScript实现设计模式（五）-- 建造者模式
date: 2019-07-16 10:24:34
tags:
- 设计模式
- TypeScript
- nodejs
categories: 设计模式
author: TinyCalf
---

> 建造者模式（Builder Pattern）:将一个复杂对象的构建与其表示分离，使得同样的构建过程可以创建不同的表示。

<!-- more -->

我们先预定义一些电脑组件以供使用，假设组装一台电脑需要主板、中央处理器、内存、机箱：
```TypeScript
abstract class Motherboard {
    public abstract toString(): string;
}
class ASUSMotherboard extends Motherboard {
    public toString(): string {
        return "ASUS Motherboard";
    }
}
class GIGAMotherboard extends Motherboard {
    public toString(): string {
        return "GIGA Motherboard";
    }
}
//...

abstract class CPU {
    public abstract toString(): string;
}
class IntelCPU extends CPU {
    public toString(): string {
        return "Intel CPU";
    }
}
class AMDCPU extends CPU {
    public toString(): string {
        return "AMD CPU";
    }
}
//...

abstract class Memory {
    public abstract toString(): string;
}
class KingstonMemory extends Memory {
    public toString(): string {
        return "Kingstin Memory";
    }
}
class CorsairMemory extends Memory {
    public toString(): string {
        return "Corsair Memory";
    }
}
//...

abstract class Crate {
    public abstract toString(): string;
}
class AigoCrate extends Crate {
    public toString(): string {
        return "Aigo Crate";
    }
}
class SamaCrate extends Crate {
    public toString(): string {
        return "Sama Crate";
    }
}
//...
```
如果我们想组装一台电脑，假设电脑类是这样的：
```TypeScript
class Computer {
    private motherboard: Motherboard;
    private cpu: CPU;
    private memory: Memory;
    private crate: Crate;
    public constructor(
        motherboard: Motherboard,
        cpu: CPU,
        memory: Memory,
        crate: Crate
    ) {
        this.motherboard = motherboard;
        this.cpu = cpu;
        this.memory = memory;
        this.crate = crate;
    }
    public boot():void {
        console.log('booting....');
        if(this.motherboard && this.cpu && this.memory) {
           console.log('success!!');
        }
    }
}
```
很好理解的一个类，构建时把组件都给Computer， 然后boot启动就好了：
```TypeScript
let motherboard = new ASUSMotherboard();
let cpu = new IntelCPU();
let memory = new KingstonMemory();
let crate = new AigoCrate();
let computer = new Computer(
    motherboard,
    cpu,
    memory,
    crate
);

computer.boot();
//stdout:
//booting....
//success!!
```
写到这里想一想有哪些问题呢？
* 第一，作为客户，我只想要一台电脑，虽然我可能知道我要的是华硕主板，金士顿的内存条，但是我并不需要知道这些零件如何构建出来；我的目的是要一台电脑，而我现在需要知道5个类，主板类、cpu类、内存类...；这显然违反了迪米特法则，也就是‘我知道的太多了’；
* 第二，上面写的这个Computer类太简单了，在构建之初就已经是可用状态，而现实中的某些类由于构建流程复杂，甚至需要异步网络请求；比如MongoClient，需要与MongoDB连接完成才处于可调用的状态，其构建过程不是constructor就能完成的。当然配一台电脑也不是一步到位的，所以我们稍微改复杂点再来解决这两个问题，Computer是一个产品所以通常会实现模板方法（关于模板方法模式见上一章），先定义一下Computer的虚类，保证Computer的扩展性：
```TypeScript
abstract class Computer {
    private motherboard: Motherboard;
    private cpu: CPU;
    private memory: Memory;
    private crate: Crate;
    private builtUp: boolean = false;
    public constructor(
        motherboard: Motherboard,
        cpu: CPU,
        memory: Memory,
        crate: Crate
    ) {
        this.motherboard = motherboard;
        this.cpu = cpu;
        this.memory = memory;
        this.crate = crate;
    }
    protected abstract buildMotherboard(): Computer;
    protected abstract buildCPU(): Computer;
    protected abstract buildMemory(): Computer;
    protected abstract buildCrate(): Computer;
    public buildup() {
        try {
            this.buildMotherboard()
                .buildCPU()
                .buildMemory()
                .buildCrate();
        } catch (e) {
            throw e;
        }
        this.builtUp = true;
    }

    public boot(): void {
        console.log("booting....");
        if (this.motherboard && this.cpu && this.memory) {
            console.log("success!!");
        }
    }
}
```
buildup是把电脑组装起来，是一个模板方法，子类只要实现其他组件的安装即可，我们来定义一台个人电脑：
```TypeScript
class PersonalComputer extends Computer {
    public constructor(
        motherboard: Motherboard,
        cpu: CPU,
        memory: Memory,
        crate: Crate
    ) {
        super(motherboard, cpu, memory, crate);
    }
    protected buildMotherboard(): PersonalComputer{
        console.log("building motherboard");
        return this;
    }
    protected buildCPU(): PersonalComputer {
        console.log("building motherboard");
        return this;
    }
    protected buildMemory(): PersonalComputer {
        console.log("building motherboard");
        return this;
    }
    protected buildCrate(): PersonalComputer {
        console.log("building motherboard");
        return this;
    }
}
```
好了，现在我们来解决刚才的问题，也就是“客户知道的太多了”的问题，相当于之前一直是客户自己来了找零件装电脑。现在来了几个新客户，进来就说“把你们这最贵的机子呈上来！”。这个时候用户只要最贵的，老板替他挑选好就行，并不需要他自己选怎么装，又因为有好几个订单，这时候怎么办？建造者模式这时候该发挥作用了，我们定义一个建造者专门处理这样的订单，考虑到以后还有其他形式的订单，所以我们先定义一个建造者虚类：
```TypeScript
abstract class ComputerBuilder {
    public abstract buildComputer(): Computer;
}
```
然后在定义一个最贵电脑的建造者：
```TypeScript
class MostExpensiveComputerBuilder {
    public buildComputer(): Computer {
        let motherboard = new ASUSMotherboard();
        let cpu = new IntelCPU();
        let memory = new KingstonMemory();
        let crate = new AigoCrate();
        let computer = new PersonalComputer(motherboard, cpu, memory, crate);
        return computer;
    }
}
```
这样一来用户接触到的就只有MostExpensiveComputerBuilder这个类了：
```Typescript
let mostExpensiveComputerBuilder = new MostExpensiveComputerBuilder();
let computer = mostExpensiveComputerBuilder.buildComputer();
computer.boot();
```
问题暂时解决了，店老板很满意，现在就要将这个模式扩展到其他订单上，比如家里老年人炒股想配台最便宜的电脑，实现方法其实和上面差不多，就不多写了。更多的还有经济型，游戏型等等， 导致电脑店现在品类越来越多了，用户又得接触很多很多builder，才知道店里有什么。为了方便用户找到自己想要的品类，我们做了一个小册子，罗列所有的品类。这个小册子就是建造者模式中的导演（Director），目前只要一个就够了，就不需要虚类了：
```TypeScript
class ShopDirector {
    private mostExpensiveComputerBuilder: ComputerBuilder
        = new MostExpensiveComputerBuilder();
    private mostCheapComputerBuilder: ComputerBuilder
        = new MostCheapComputerBuilder();
    private gamePlayComputerBuilder: ComputerBuilder
        = new GamePlayComputerBuilder();

    public getMostExpensiveComputer(): Computer {
        return this.mostExpensiveComputerBuilder.buildComputer();
    }
    public getMostCheapComputer(): Computer {
        return this.mostCheapComputerBuilder.buildComputer();
    }
    public getGamePlayComputer(): Computer {
        return this.gamePlayComputerBuilder.buildComputer();
    }
}
```
好啦，这就是整个建造者模式了。针对构建比较复杂的对象，构建顺序或者构建方法等最终会影响构建结果，所以我们需要增加一个建造者以防止模块外部直接接触对象。当建造者变多的时候我们可能又需要一个Director来约束建造者。
相比于工厂方法模式，最大的区别就是工厂模式中的“产品”是简单构建的对象，不存在构建方法不同会影响结果。
相比于模板方法模式，建造者模式向 “用户”输出的是一个模块，该模块中的方法可以直接调用以获得各种“产品”；而模板方法模式向“用户”输出的是一个模板，“用户”通过继承这个模板，并简单了解上层构建过程以实现自己的类。简单说就是建造者模式的“用户”在调用方法，而模板方法模式的“用户”在继承类。
参考了几篇文章写的建造者模式，其实根据假设场景的不用，具体实现也都不太一样，但是核心是不变的，就是定义中的“将复杂对象的构建与其表示分离”。
建造者模式就写到这。

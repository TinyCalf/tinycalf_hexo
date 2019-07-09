---
title: TypeScript实现设计模式（六）-- 代理模式
date: 2019-07-23 10:18:02
tags:
- 设计模式
- TypeScript
- nodejs
categories: 设计模式
author: TinyCalf
---
> 代理模式（Proxy Pattern）：为其他对象提供一种代理以控制对这个对象的访问

<!-- more -->

假设我们要实现一个用户管理接口：
```typescript
/**
 * @file IUserManager.ts
 */
export default interface IUserManager {
    getUser(name: string): string;
    addUser(name: string): string;
    delUser(name: string): string;
    getUserList():string[];
}
```
然后实现一下真实用户管理类：
```typescript
/**
 * @file UserManager.ts
 */
import IUserManager from "./IUserManager";

export default class UserManager implements IUserManager {
    private static userList: string[];

    public constructor() {
        UserManager.userList = [];
        UserManager.userList.push("Tom");
        UserManager.userList.push("Jacob");
        UserManager.userList.push("Jerry");
    }

    public getUser(name: string): string {
        let index = UserManager.userList.indexOf(name);
        if (index != -1) {
            return UserManager.userList[index];
        } else {
            throw new Error(`Can not find ${name}.`);
        }
    }

    public addUser(name: string):string {
        let index = UserManager.userList.indexOf(name);
        if (index != -1) {
            throw new Error(`${name} has already exists.`);
        } else {
            UserManager.userList.push(name);
            return name;
        }
    }

    public delUser(name: string):string {
        let index = UserManager.userList.indexOf(name);
        if (index != -1) {
            UserManager.userList.splice(index, 1);
            return name;
        } else {
            throw new Error(`can not find ${name}`);
        }
    }

    public getUserList(): string[] {
        return UserManager.userList;
    }
}
```
代理类形式很简单，我们就不一步步推演了；下面我们就实现一个代理：
```typescript
/**
 * @file UserManagerProxy
 */
import IUserManager from './IUserManager';
import UserManager from './UserManager';

export default class UserManagerProxy implements IUserManager {
    private userManager:IUserManager;

    public constructor(_userManager: IUserManager) {
        this.userManager = _userManager;
    }

    public getUser(name: string):string {
        return this.userManager.getUser(name);
    }

    public addUser(name: string):string {
        return this.userManager.addUser(name);
    }

    public delUser(name: string):string {
        return this.userManager.delUser(name);
    }

    public getUserList():string[] {
        return this.userManager.getUserList();
    }
}
```
可以看到代理类和被代理类都是对IUserManager的实现，代理实例对帮助被代理实例执行其方法，实现访问控制；我们再看下标准场景类：
```typescript
/**
 * @file client.ts
 */
import UserManager from './UserManager';
import UserManagerProxy from './UserManagerProxy';

let userManager = new UserManager();
let userManagerProxy = new UserManagerProxy(userManager);

let userList = userManagerProxy.getUserList();
console.debug(userList);
//stdout: [ 'Tom', 'Jacob', 'Jerry' ]
```
ok, 这就是最简单的代理模式，用一个proxy实例代替正真的产品去执行其方法。但是在这个例子里好像看不出代理模式有什么用呀？好的，那么我们加点料。用户管理是个复杂的系统，针对用户操作，我们首先要确认执行者有没有权限，在执行成功后记录管理员操作日志。这些代码写在哪里？自然是写在代理中，被代理的实例本身只需要关注最原始的功能，让代理执行其他复杂的处理流程。想象一下如果不用模式你会怎么做？一个方法里先写了权限判断，再写逻辑主体，最后再写操作日志。这完全不符合单一职责原则，而代理模式能很好的解决这个麻烦：
```typescript
/**
 * @file UserManagerProxy
 */
import IUserManager from './IUserManager';
//import UserManager from './UserManager';

export default class UserManagerProxy implements IUserManager {
    private userManager:IUserManager;

    public constructor(_userManager: IUserManager) {
        this.userManager = _userManager;
    }

    public getUser(name: string):string {
        console.log("正在检查权限");
        return this.userManager.getUser(name);
    }

    public addUser(name: string):string {
        console.log("正在检查权限");
        return this.userManager.addUser(name);
        console.log("正在记录操作日志");
    }

    public delUser(name: string):string {
        console.log("正在检查权限");
        return this.userManager.delUser(name);
        console.log("正在记录操作日志");
    }

    public getUserList():string[] {
        console.log("正在检查权限");
        return this.userManager.getUserList();
    }
}
```
还是按照原来的方式执行，结果如下：
```bash
正在检查权限
正在记录操作日志
[ 'Tom', 'Jacob', 'Jerry' ]
```
代理模式自然根据具体情况有各种扩展和变种。我暂时不一一列举了，如果项目里有比较好的实践，我会再回来写一写。这里我再写一种比较复杂的代理形式。假设UserManager需要扩展很多方法， 按照现在的方式，如果我要求每个方法都要检查权限和写操作日志， 那代理类需要把所有方法实现一遍，并加上检查权限和写操作日志。这种形式可能太麻烦了，所以我们要实现动态代理，也就是实现动态获取实例的方法并代理其执行。这点在js中倒是非常好实现。首先我们要脱离ts， 用js写一个动态代理类，这个动态代理可以代理所有的类：
```javascript
/**
 * @file DynamicProxy
 */
/* jshint esversion: 6 */
function DynamicProxy(_class, _handler) {
    let proxy = new _class();
    for (let funcName in _class.prototype) {
        if(typeof _class.prototype[funcName] == 'function')
        proxy[funcName] = function() {
           return _handler.invoke(_class.prototype[funcName], arguments);
        };
    }
    return proxy;
}

module.exports = DynamicProxy;
```
这个类接收两个参数，一个是class，以方便我们找到类中的所有方法，另一个是handler，我们后面会实现一下，handler中可以定义“要怎么代理”，也就是执行正真的方法之前和之后要干什么。我们向proxy变量上面绑定了class中的所有方法，并通过handler中定义的方式执行，这就是动态代理啦。当然我们是在ts中使用，我们我们要定义一下申明文件：

```typescript
/**
  * @file DynamicProxy.d.ts
  */
import InvocationHandler from './InvocationHandler';
declare function DynamicProxy<T>(_class: new()=> T, handler: InvocationHandler): T;

export default DynamicProxy;
```
接着定义handler，增加权限判断和操作日志：
```typescript
/**
  * @file UserManagerIH..ts
  */
import UserManager from "./UserManager";
import InvocationHandler from './InvocationHandler';

export default class UserManagerIH extends InvocationHandler {
    private userManager: UserManager;

    public constructor(_userManager: UserManager) {
        this.userManager = _userManager;
    }

    public invoke(method: Function, args: any[]) {
        console.log("正在检查权限");
        let result = method.apply(this.userManager, args);
        console.log("正在记录操作日志");
        return result;
    }
}
```
IH表示InvocationHandler，这个抽象类就一个invoke方法，自己实现一下吧。好啦，method.apply就是在执行真正的方法，前后加上了检查权限和操作记录。如果你还需要方法名来区分方法之间的微小区别，这里也可以加一个funcName参数。我举得例子现在还不一定是最佳方案，但是大致形式是这样的，一个动态代理动态绑定所有方法，一个IH定义各种处理流程。ok， 我们最后来看场景：
```typescript
/**
 * @file client.ts
 */
import UserManager from './UserManager';
import DynamicProxy from './DynamicProxy';
import UserManagerIH from './UserManagerIH';


let userManager = new UserManager();

let handler = new UserManagerIH(userManager);

let proxy = DynamicProxy(UserManager, handler);

let userList = proxy.getUserList();
console.debug(userList);
// stdout:
//正在检查权限
//正在记录操作日志
//[ 'Tom', 'Jacob', 'Jerry' ]
```
神奇吧，我并没有定义一个代理类把被代理对象的方法一个一个实现一遍，但是可以通过动态代理调用所有方法这就是动态代理。到这里一般都会提到面向切面编程（AOP，有兴趣可以查下），具体业务逻辑往往是分层的，就比如我们的UserManager， 需要权限判断，需要写日志，甚至需要通知，这里的每一层都是一个切面；如果每个方法都把这些曾都实现一遍，想想就觉得乱套，而且不仅仅是UserManager如何，其他什么什么Manager也要实现；当你实现了上百个接口以后，突然要改通接口，想想有多痛苦吧。代理模式就是一个很好的解决方案，让各个分层职责清晰，扩展性高。

代理模式的形式其实多种多样，应用也十分广泛，如果看到各种源码中有比较好的例子，我会再回来补充。

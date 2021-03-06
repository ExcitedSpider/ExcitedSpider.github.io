---

layout: post
title: "TypeScript 3.8 新特性"
date: 2020-3-23
excerpt: "演进"
tags: [TypeScript]
comments: false

---

github 收购了 npm，微软在前端生态界又多了一个利器啊，这下 TypeScript+Edge Chrome+npm，微软在前端的话语权不可说不高了。没想到当年因为 ie 被喷的体无完肤的微软，又在前端领域当了领军者。

## 仅类型导入导出

新增仅类型导入导出语法(Type-Only Imports and Exports)

```typescript
import type { SomeThing } from "./some-module.js";
export type { SomeThing };
```

这个语法可以将同名的类型(type)和实例(instance)分开来，表示仅导出类型。
在 TypeScript 中，很多类型是由 d.ts 声明文件写的，导致的一个问题就是类型和实例名称一致。在使用时，会导致混淆。使用`import type`就可以避免这个问题，举个官网的例子：

```typescript
import type { Component } from "react";

interface ButtonProps {
  // ...
}

class Button extends Component<ButtonProps> {
  //               ~~~~~~~~~
  // error! 'Component' only refers to a type, but is being used as a value here.
  // ...
}
```

这里仅将作为interface的Component导入，而不将同名class的Component导入，就可以限制其使用。

## ECMAScript 私有成员

OOP的一个常见特性就是给类添加私有成员，外部不可访问。以前JS要做私有成员一般会通过一些比较诡吊的方式，比如用Symbol。现在TS3.8实现了ECMAScript stage 3的标准私有成员语法`#`。

```typescript
class Person {
    #name: string

    constructor(name: string) {
        this.#name = name;
    }

    greet() {
        console.log(`Hello, my name is ${this.#name}!`);
    }
}

let jeremy = new Person("Jeremy Bearimy");

jeremy.#name
//     ~~~~~
// Property '#name' is not accessible outside class 'Person'
// because it has a private identifier.
```

但是这个语法和TS之前的`public`，`private`修饰符冲突，所以在开发中最好统一一种语法。我的看法是最好用现在的`#`，毕竟是ECMAScript标准。

`#`私有成员语法相较非标准的`private`更加严格，声明的私有成员甚至不能被探测(detected)到，且对包含的class独立，不能被子类重写(overwrite)，这种特性被称为hard privacy。

```typescript
class C {
    #foo = 10;

    cHelper() {
        return this.#foo;
    }
}

class D extends C {
    #foo = 20;

    dHelper() {
        return this.#foo;
    }
}

let instance = new D();
// 'this.#foo' refers to a different field within each class.
console.log(instance.cHelper()); // prints '10'
console.log(instance.dHelper()); // prints '20'
```

但是对我个人来说，其实很少大规模使用OOP特性来组织代码，更希望能够多一些FP的特性。

## export * as ns

一个用于给全部导出成员起一个namespace的语法糖

```typescript
import * as utilities from "./utilities.js";
export { utilities };
```

## Top-Level await

在全局上下文时可以使用await语法，就像在async函数中一样。

```typescript
const response = await fetch("...");
```

还不了解async/await异步语法的同学可以参考阮老师的es6标准入门<a herf="https://es6.ruanyifeng.com/?search=await&x=0&y=0#docs/async">相关章节</a>

## ES2020标准支持

typescript可以选择编译目标(target)为es2020，支持各种语法。对于写nodejs的同学和学习es2020标准的同学可能会有一些帮助，对于我们前端来说编译目标还是越低越好，所以不再多说。

## 优化Linux上的Lang Server文件监控性能

这个我是比较有体会，有段时间非常喜欢折腾linux做开发环境，玩过ubuntu，deepin，manjaro之类的发行版。但是Lang Server的表现太差了，初始化需要的时间很长，占内存很高。3.8的TypeScript专门对此作了优化，不过实际情况怎么样我还没有尝试，因为手上暂时还没有linux桌面环境。
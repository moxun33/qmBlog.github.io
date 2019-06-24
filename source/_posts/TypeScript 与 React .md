---
layout: typescript
title: TypeScript 与 React  
date: 2018-07-28 00:26:52
tags:
    - React
    - TypeScript
---

最近开发工作缓下来了， 就抽时间看看传说中的神器 TypeScript。主要描述一些 TypeScript 在 React 项目中的一些使用方法。
<!-- more -->

# 一、TypeScript 是什么

TypeScript 是 JavaScript 类型的超集，它可以编译成纯 JavaScipt。TypeScript可以在任何浏览器、任何计算机和任何操作系统中运行， 并且它是开源 的。

# 二、 TypeScript 的基础知识

1、 TypeScript 的基本类型：TypeScript支持与JavaScript几乎相同的数据类型，此外还提供了实用的元组、枚举类型方便我们使用。如：``let exist: boolean = false``

2、TypeScript 的变量声明：``let``和``const``是JavaScript里相对较新的变量声明方式。 像我们之前提到过的， let在很多方面与var是相似的，但是可以帮助大家避免在JavaScript里常见一些问题。 ``const``是对``let``的一个增强，它能阻止对一个变量再次赋值。这跟 JavaScript 的其他变量用法几乎一模一样。

3、 TypeScript 的接口：TypeScript的核心原则之一是对值所具有的结构进行类型检查。 在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。接口中属性的顺序不影响接口的判断。

接口的定义：

```php
interface IPerson {
  name: string;
  age? number;
  readonly gender: string; 
}
```

在属性后加上``？``表示该属性的可选的，否则的必要条件; 在属性名前加上``readonly``表示实现该接口后（对象字面量构造），该属性只读，不能写入新的值。

接口的对象构造

```php
const me: IPerson = {name: "qimajiang", age: 25, gender: "男"}
```
跟变量的声明差不多， 只是类型是自定义的。

4、TypeScript 的类：传统的JavaScript程序使用函数和基于原型的继承来创建可重用的组件，但对于熟悉使用面向对象方式的程序员来讲就有些棘手，因为他们用的是基于类的继承并且对象是由类构建出来的。 从ECMAScript 2015，也就是ECMAScript 6开始，JavaScript程序员将能够使用基于类的面向对象的方式。 使用TypeScript，我们允许开发者现在就使用这些特性，并且编译后的JavaScript可以在所有主流浏览器和平台上运行，而不需要等到下个JavaScript版本。

类的基础用法

```php

class Person {
    name: string;
    constructor(name: string) {
        this.name = name;
    }
    say() {
        return "Hello, " + this.name;
    }
}

let person = new Person("qimajiang");

const sayHello = person.say(); // Hello, qimajiang
```

类的继承用法， 跟 Java 或其他编程语言一样， 类是可以继承的。用法如下：

```php

class Man extends Person {
    constructor(name: string) {
        super(name)
    }
    getGender() {
        console.log('I am a man');
    }
}

const man = new Man("qimajaing")
const manSay = man.say(); 
const gender = man.getGender();
```

TypeScript 的类中也可以使用修饰符来限制类中属性或方法的访问。 修饰符有： ``public``、``private``、``protected``、``readonly``

关于类的更多知识， 访问 <https://www.tslang.cn/docs/handbook/classes.html>

5、TypeScript的函数：和JavaScript一样，TypeScript函数可以创建有名字的函数和匿名函数。 你可以随意选择适合应用程序的方式，不论是定义一系列API函数还是只使用一次的函数。在 TypeScript 中的参数、返回值、中间变量都可以指定类型。这样让编写的函数更加安全稳定健壮、在 IDE 中还可以智能提示，加快编码效率，降低错误率。如：

```php
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };

```

在TypeScript里我们可以在参数名旁使用 ``?``实现可选参数的功能; 默认参数、剩余参数、箭头函数或其他用法与 在JavaScript 中的一样。

### 到目前为止，主要学习了 TypeScript 的最基础知识，足够创建一些最基本的 web 应用了。至于 TypeScript 的其他语法：泛型、枚举等等，就不再一一展开了，可访问<https://www.tslang.cn/docs/handbook/>查看.


# 三、 JSX

## 1、JSX 知识

JSX是一种嵌入式的类似XML的语法。 它可以被转换成合法的JavaScript，尽管转换的语义是依据不同的实现而定的。 JSX因 ``React``框架而流行，但是也被其它应用所使用。 TypeScript支持内嵌，类型检查和将JSX直接编译为JavaScript。想在 TypeScript 使用 ``jsx``，必须做到：

> 1、给文件一个 ``.tsx``扩展名
> 2、启用 ``jsx``选项

TypeScript具有三种JSX模式：`` preserve``， ``react``和 ``react-native``。 这些模式只在代码生成阶段起作用 - 类型检查并不受影响。 在 ``preserve``模式下生成代码中会保留JSX以供后续的转换操作使用（比如： ``Babel``）。 另外，输出文件会带有 ``.jsx``扩展名。 ``react``模式会生成 ``React.createElement``，在使用前不需要再进行转换操作了，输出文件的扩展名为 ``.js``。 ``react-native``相当于 ``preserve``，它也保留了所有的JSX，但是输出文件的扩展名是 .js。


## 2、TypeScript 与 React 的整合

要想一起使用JSX和React，要使用 [React类型定义](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)。 这些类型声明定义了 JSX合适命名空间来使用React。

# 四、创建TypeScript风格的React应用

## 1、 安装 TypeScript 版本的定义依赖包：
（1）在原有的 React 项目，除了安装 ``react``，还需要安装``@types/react``,若其他 react 库存在@types 定义包， 也一并下载安装。

（2）如果是使用 create-react-app 创建 React 项目， 可以在创建时增加参数``--scripts-version=react-scripts-ts``，就好创建 TS 风格的 React 项目了。

## 2、 weback 配置

至少要增加 ts 的配置 `` { test: /\.tsx?$/, loader: "ts-loader" },``


## 2、注意事项

（1）在普通的 React项目中，我一般这样引用 React，``import React from 'react'``,但在 TS 风格的 React 项目中， 需要这样``import * as React from 'react'``

（2） 创建无状态组件也有区别，具体分析下一篇展开。

（3） TS 风格 React 项目中，组件的状态和属性定义，尽可能用``Interface``声明。

(4)  要习惯对每个变量声明数据类型，减少出错概率，也方便 Debug。

# 五、总结

本人主要学习了 TypeScrip 的一些常用特性， 以及如何创建一个 TS 风格的 React 项目。更多的TS 实战，后面会结合项目开发记录下来。








---
title: Why has Class-MetaClass design in Objective-C？
date: 2020-04-25 10:10:01
categories: ["iOS", "基础理论"]
tags: ["2020"]
comments: true
---

为什么Objective-C中有Class和MetaClass这种设计？

这个问题某日在[掘金](https://juejin.im/entry/59bb8b895188257e70531bf9)上看到的。我认为这不是一个技术领域的问题，而是编程语言的设计选择领域的问题。

在Objective-C编程语言的设计中，类（Class）既是一个用于描述对象实例（object）的属性和行为的工具，也是一个对象（Object），有自身的属性和行为；那类（Class）的属性和行为使用什么来描述呢？答案就是：元类（MetaClass）。只不过元类（MetaClass）处于编程语言的实现底层，对开发者是透明的。

那取消掉元类（MetaClass）可以吗？

答案是：可以。但是这需要从该编程语言的设计上做根本的调整：取消掉“类（Class）也是对象（Object）”的设计，把类（Class）同时当作描述对象实例（object）和类（Class）自身的属性和行为的工具。采用与此类似的设计的编程语言也存在，比如C++。


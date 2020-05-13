---
title: Why has Class-MetaClass design in Objective-C？
date: 2020-05-05 10:10:01
categories: ["基础理论"]
tags: ["2020"]
comments: true
---

## 前言
在面向对象领域中，方法（method）和函数（function）有什么区别和关联呢？


## 方法和函数的区别
在面向对象领域中，方法和函数的区别是：

1. 定义不一样

    - 函数是指一段可以直接被其名称调用的代码块
    - 方法指的是一段被它关联的对象通过它的名字调用的代码块
    
1. 与对象的关系不一样

    - 函数独立于对象
    - 方法依附在对象之上，可以在代码块内直接处理对象上的成员数据

1. 传递的数据（比如参数）不一样

    - 传递给函数的数据都是明文明确的
    - 传递给方法的数据有部分是隐式的，其中隐式部分的数据主要是调用该方法的对象实例
    
1. 可访问范围不一样

    - 一般来说（在不考虑module、package等作用域的设计情况下），函数的可访问范围是全局性的，即可以在代码的任何地方访问到
    - 方法的可访问范围由其访问修饰符决定，基本上其范围都局限在所依赖的对象内

<!-- more -->

## 方法和函数的关联  
在面向对象领域中，方法和函数的关联是：在概念上，方法可简单视作函数的特例——方法就是对象的函数。

## C++标准为何没有使用“方法”而是使用“成员函数”

C++标准为何没有使用“方法”这个术语而是选择使用了“成员函数”这个术语呢？

这个问题没有正式的、官方的答案。我的个人理解则是：

- 这可能和编程语言设计观相关：对于C++
作者来说，采用“成员函数”可以减少一个术语概念，降低复杂性
- 这也可能和编程语言的底层实现的设计相关：C++ 的一个关键目标是尽量地节省额外开销，特别是内存的开销，为此，在 C++ 代码和硬件之间，C++ 没有做额外的抽象、虚拟或者其他的数学模型，而是直接把基本类型（char, int, double 等）映射到内存中的实体——比如字节（Byte）、字（Word），把有类型的对象映射到内存中一块连续空间内。在处理类的方法（此处使用面向对象术语）时，并没有做额外抽象，而是和处理普通函数一样：做直接的内存映射（在内存上都是一段连续的指令）。那么，在底层实现这个层面上，以及没有做额外的抽象、虚拟的情况下，类的方法和普通函数没有本质区别，使用“成员函数”的术语似乎优于“方法”这个术语了。

  > 关于C++ 的类的一个微妙问题：
  >
  > 一个常见的混淆其实只是一个微妙的术语问题：由于它的演化来自C，在C++ 中的术语对象和C语言一样是意味着存储器区域，而不是类的实体，在其它绝大多数的面向对象语言也是如此。举例来说，在C和C++中，语句int i;定义一个int类型的对象，这就是变量的值i将在指派时，所存入的存储器区域。
  > 
  > 源自：[维基 -《C++》]( https://zh.wikipedia.org/wiki/C%2B%2B#C++中的特色)


## 参考资料

- [stackoverflow -《What's the difference between a method and a function?》](https://stackoverflow.com/questions/155609/whats-the-difference-between-a-method-and-a-function?page=1&tab=votes#tab-top)
- [《方法和函数的区别》](https://blog.csdn.net/notsaltedfish/article/details/75174556)
- [《方法（method）和函数（function）有什么区别？》](https://www.cnblogs.com/wancy86/p/7271850.html)
- [维基 -《C++》]( https://zh.wikipedia.org/wiki/C%2B%2B#C++中的特色)
- [《C++ 的几个基本原理和技术》](https://liam.page/2017/04/09/Foundations-of-Cpp/)
- [《Foundations of C++（PPT版本）》](http://cs.ioc.ee/etaps12/invited/stroustrup-slides.pdf)
- [《Foundations of C++（论文版本）》](http://www.stroustrup.com/ETAPS-corrected-draft.pdf)


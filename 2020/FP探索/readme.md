---
title: FP探索
date: 2020-07-01 12:34:56
categories: ["技术"]
tags: ["2020", "基础理论", "Review", "非终稿"]
comments: true
---

## 前言

本文主要是对计算机科学领域中的函数式编程（英语：Functional Programming）范式的理论、特性及特性的实现方案进行探索。

<!-- more -->

## 如何实现 Fist-Class Function 
函数式编程范式的核心思想是：`Fist-Class Function`，即函数作为一等公民，可以作为别的函数的参数、函数的返回值，赋值给变量或存储在数据结构中。

实现`Fist-Class Function`的方案主要有3种：

- 指针（Pointer）

    - 实现：使用函数指针代替函数进行一等公民的相关操作（入参、赋值等）。
    
    - 优点：实现简单。
    
    - 缺点：只适合支持指针的编程语言；函数需要提前定义且无法直接访问函数外部环境（指针变量定义处的环境）的局部变量。

- 函数对象（Function Object）

    - 实现：使用对象模拟函数，然后使用对象代替函数进行一等公民的相关操作（入参、赋值等）。
    
    - 优点：实现简单；可携带状态。
    
    - 缺点：函数需要提前定义且无法直接访问函数外部环境（函数对象变量定义处的环境）的局部变量（对于不支持匿名类方式的编程语言）；代码形式上和函数定义相差较多。

- 闭包（Closure）

    - 实现：构造一个名为闭包的数据结构实体存储函数的指针和函数依赖的环境（环境的作用是为函数中的自由变量提供绑定信息），然后使用闭包代替函数进行一等公民的相关操作（入参、赋值等）。
    
    - 优点：函数可以在使用时才定义；函数可直接访问函数外部环境（闭包定义处的环境）的局部变量；代码形式上和函数定义相近；闭包由编译器在词法分析阶段自动构造生成（指针和函数对象需要手动构造）
    
    - 缺点：实现相对复杂，需要编程语言的编译器在词法闭包解析的操作上增加为闭包增加“捕获”变量的特性。

## 实现方案之函数对象（Function Object）
函数对象的设计思想是：构造一个具备函数特征的对象代替普通函数，然后像调用普通函数那样调用对象。

而上述中所谓“函数特征”，其定义和实现由具体由语言设计者确定，一般来说，该特征表现为：（基于类的编程语言中）对象需要实现或者`override`指定的方法。

比如，在C++ 中，函数对象由`override`了运算符方法`operator()`的类表示：

``` C++
#include <iostream>     // std::cout
#include <algorithm>    // std::sort
#include <vector>       // std::vector
#include <iterator>     // std::ostream_iterator

// comparator predicate: returns true if a < b, false otherwise
struct IntComparator
{
  bool operator()(const int &a, const int &b) const
  {
    return a < b;
  }
};

int main()
{
    std::vector<int> items { 4, 3, 1, 2 };
    std::sort(items.begin(), items.end(), IntComparator());

    std::copy(items.begin(), items.end(), std::ostream_iterator<int>(std::cout, " ")); // -> 1 2 3 4

    return 0;
}
```

又比如，在Java 中，函数对象由只有一个方法的接口（比如最常见的`Comparator Interface`、`Callable Interface`和`Runnable Interface`等）表示，该接口在实现上多表现为匿名类：

``` Java
import java.util.Arrays;
import java.util.Comparator;

class Main {
  public static void main(String[] args) {
    Integer[] array = {4, 3, 1, 2};
    Comparator<Integer> numComparator = new Comparator<Integer>() {
        public int compare(Integer num1, Integer num2) {
            return num1.compareTo(num2);
        }
    };
    Arrays.sort(array, numComparator);
    
    System.out.println(Arrays.toString(array)); // -> [1, 2, 3, 4]
  }
}
```

**参考资料：**
- [WiKi-《Function object》](https://en.wikipedia.org/wiki/Function_object)

## 实现方案之闭包（Closure）

> 下面对闭包的描述抛开了闭包的发展历史和闭包在数学领域的概念，仅仅是从计算机科学的工程角度进行描述。

函数作为一等公民，要像整数、字符串这些普通数据一样在使用时才进行定义，就必须要解决一个问题：自由变量的绑定的问题。具体来说，就是一个函数创建后，其使用了一个定义在函数体外的变量（即自由变量），当该函数脱离当前环境后，传递到另一个环境运行时，该如何获得这个自由变量关联的数据？

> 自由变量是指：函数体中的除了参数和局部变量之外的变量；简单说就是定义在函数体外的变量。
>
> 绑定是指：把标识符与实体（数据和/或代码）关联起来。
>
> 自由变量的绑定是指：把代表自由变量的标识符和数据关联起来，简单理解就是给自由变量指定值

下面将通过一个简化的场景例子对该问题进行详细说明：

```
// 假设存在一个支持运行下列的伪代码的计算系统
// 假设当前计算系统支持直接传递函数传递到不同作用域
// 假设当前计算系统存在有2个作用域：SCOPE-1 和 SCOPE-2

+--------------------+   
| // SCOPE-1         |  
|                    |
| y = 1;             |
| f(x) {             |
|   retrun x + y;    |
| };                 |
|                    |
| f(1); // -> 2      |
+--------------------+


+--------------------+   
| // SCOPE-2         |  
|                    |
|                    |
|                    |
+--------------------+
```

在`SCOPE-1`中，存在一个局部变量`y`和一个函数`f`，`y`是`f`的自由变量，在离开`SCOPE-1`后，`y`就不存在了——这意味着函数`f`传递到`SCOPE-2`后，会因无法确定`y`绑定的值，从而导致运算出错。

```
+--------------------+   
| // SCOPE-1         |  
|                    |
| y = 1;             |
|                    |
| +----------------+ |
| |f(x) {          | |
| |  retrun x + y; |-|----\
| |};              | |    |
| +----------------+ |    |
|                    |    |
| f(1); // -> 2      |    |
+--------------------+    |
                          |
                          | pass function f
                          |
+--------------------+    |   
| // SCOPE-2         |    |  
|                    |<---/
| f(1); // -> error  |
|                    |
+--------------------+

```

那么该如何解决呢？把函数`f`和自由变量`y`打包一起，传递到`SCOPE-2`：

```
+--------------------+   
| // SCOPE-1         |  
|                    |
| +----------------+ |
| |y = 1;          | |
| |f(x) {          | |
| |  retrun x + y; |-|----\
| |};              | |    |
| +----------------+ |    |
|                    |    |
| f(1); // -> 2      |    |
+--------------------+    |
                          |
                          | pass closure "f"
                          |
+--------------------+    |   
| // SCOPE-2         |    |  
|                    |<---/
| f(1); // -> 2      |
|                    |
+--------------------+

```

实现这种打包的技术称为闭包，而同时打包的产物也称之为闭包。

在具体实现上，闭包的技术方案大致如下：

1. 构造一个环境，存储自由变量的绑定信息——这在术语上，又称自由变量被“捕获”了；

    > - 绑定信息包括自由变量的符号和自由变量的值，比如`{y: 2}`；
    > - 存储自由变量的值的方式可以是值复制也可以是引用，具体由语言设计者确定；

2. 构造一个数据结构实体，存储函数的指针和该环境——这个数据结构实体就是闭包；
3. 对函数进行转换处理，包括把闭包作为其参数，以及把函数体中的自由变量转化为局部变量（局部变量的值根据自由变量的符号从环境中获得）；

对于支持闭包的编程语言，以上的步骤由编译器在进行词法分析时自动完成，无需手动操作。

如果手动实现，怎么实现呢？下面使用C语言模拟实现：

```c
#include <stdio.h>

struct _closure_f_env {
    int y;
};

struct _closure_f {
    void *fp;
    struct _closure_f_env env;
};

/**
f(x) {
  return x + y;
}
*/
static int _f_impl(struct _closure_f *cp, int x) {
    int y = cp->env.y;
    return x + y;
}

struct _closure_f scope_1() {
    int y = 1;
    struct _closure_f_env env = {y};
    struct _closure_f f = {&_f_impl, env};

    int (*fp)(struct _closure_f *, int) = f.fp;
    int result = fp(&f, 1);
    printf("\nscope_1: ");
    printf("%d\n", result); // -> 2
    return f;
}

void scope_2(struct _closure_f f) {
    int (*fp)(struct _closure_f *, int) = f.fp;
    int result = fp(&f, 1);
    printf("\nscope_2: ");
    printf("%d\n", result); // -> 2
}

int main(void) {
    struct _closure_f f = scope_1();
    scope_2(f);
}
```

**参考资料：**

- [《【闭包】你真的理解闭包和lambda表达式吗》](https://www.jianshu.com/p/c22db2a91989)
- [《Closure (computer programming)》](https://en.wikipedia.org/wiki/Closure_(computer_programming))
- [《再谈闭包》](https://lotabout.me/2016/thoughts-of-closure/)
- [《关于编程语言中的闭包》](http://yungkcx.github.io/jekyll/update/2017/02/07/About-Closure.html)
- [《Closure conversion: How to compile lambda》](http://matt.might.net/articles/closure-conversion/)
- [《Lambda Calculus And Closure》](https://www.kimsereylam.com/racket/lisp/2019/02/06/lambda-calculus-and-closure.html)
- [《Paper: The Function of FUNCTION in LISP》](http://banjiewen.net/the-function-of-function-in-lisp.html)
- [《Lambda and Lexical Scope (Hunk M)》](https://www.cs.utexas.edu/ftp/garbage/cs345/schintro-v14/schintro_64.html)
- [《A Tutorial Introduction to the Lambda Calculus》](https://personal.utdallas.edu/~gupta/courses/apl/lambda.pdf)
- [《CS 611
Advanced Programming Languages: Lecture 7: Lambda calculus（PPT）》](http://www.cs.cornell.edu/courses/cs611/2000fa/slides/lec07.pdf)
- [《编程语言的基石——Lambda calculus》](https://liujiacai.net/blog/2014/10/12/lambda-calculus-introduction/)

### lambda 与 closure 的区别
在编程语言领域，从底层实现层面上看：

- lambda等同匿名函数

  > 对于支持lambda的编程语言来讲，lambda第一语义是：支持使用lambda表达式生成匿名函数，lambda第二语义是：使用lambda代称匿名函数

- closure是由函数和其运行环境构成的实体

在编程语言领域，从应用层面上看，由于支持closure的编程语言一般也支持lambda，创建closure时多使用lambda的方式，所以在这二者经常一起出现的情况下，为了简化概念，把closure等同lambda，未尝不可。

### 柯里化（Currying）
在数学角度，柯里化就是一个逐次消元的过程。当把函数的元全消掉，就得到了值。值就是零元函数，比如：

```
二元函数：
f(x,y)=x+y

在x=1时，带入得：
g(y)=f(1,y)=1+y
```

在编程角度，柯里化可用于把一个多参函数转换为由多个单参函数组成的连锁函数，比如在JavaScript中：

```js
// f(x, y) = x + y
var fxy = function(x) {
  return function(y) {
    return x+y;
  }
}
console.log("f(1,2) = ", fxy(1)(2)); // -> f(1,2) = 3 

```

**参考资料：**

- [《如何理解functional programming里的currying与partial application?》](https://www.zhihu.com/question/30097211)
- [WiKi-《柯里化》](https://zh.wikipedia.org/wiki/柯里化)




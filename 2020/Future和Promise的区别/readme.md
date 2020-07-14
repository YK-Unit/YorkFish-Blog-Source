---
title: Future和Promise的区别
date: 2020-05-31 10:10:01
categories: ["技术"]
tags: ["2020", "基础理论"]
comments: true
---

## 前言

Future模型和Promise模型是并发编程领域中的两种相近的异步编程模型，Future和Promise则是并发编程语言分别对Future模型和Promise模型进行实现后的产物。

Future和Promise之间，既有相同之处，也有不同之处。

<!-- more -->

## Future和Promise的相同点

Future和Promise的设计理念是一致的：作为一个异步任务的运行结果的占位符对象供开发者使用。

> A future or promise represents the future value of an asynchronous task.

Future和Promise给开发者带来的好处是相同的：

- 简化了异步编程，让开发者“摆脱”对线程的依赖和关注，只专注于异步任务（异步业务代码）的编码即可

  > - 【让开发者“摆脱”对线程的依赖和关注】这个是相对传统的“多线程模型”而言。在传统的“多线程模型”中，编写异步业务代码，开发者需要手动创建线程，若需要高效的并发性能，则还需要开发者通过线程池等手段手动管理线程
  > - 事实上，进行了高级抽象的异步编程模型都有这种好处，比如Objective-C中的GCD、NSOperation。
  
- 提供了更优雅、好用的异步编程，比如：异步代码同步化，链式调用，任务组合，摆脱回调地狱等

## Future和Promise的不同点

Future和Promise的区别是：

- 状态不一样

    Future的状态有2种：
    
    - uncompleted：未完成状态，Future的初始化状态
    - completed：完成状态，有可能操作成功也可能操作失败
      
      > 其中，completed根据结果又分为2种：
      > - completing with a value
      > - completing with an error
    
    Promise的状态有3种：
    
    - pending：待定状态，Promise的初始化状态
    - fulfilled：满足状态，意味着操作成功
    - rejected：拒绝状态，意味着操作失败
    
- 状态修改机制不一样

    - Future的状态由内部自行管理：当异步任务执行完成或者执行过程中抛出错误，一个Future就自动从uncompleted状态变为completed状态
    - Promise的状态由外部开发者手动管理：开发者根据控制流逻辑，调用指定函数（`fulfill`、`reject`）来显式修改的一个Promise的状态

    > - `fulfill`函数的作用是：修改Promise的状态为fulfilled，同时返回结果值
    > - `reject`函数的作用是：修改Promise的状态为rejected，同时返回错误信息
    > 在支持Promise的不同编程语言中，`fulfill`、`reject`函数可能有其他名称，比如在JavaScript中，二者对应叫`resolve`和`reject`

- 结果返回机制不一样

   - Future通过return方式返回结果
   
   - Promise通过指定函数（`fulfill`、`reject`接口）返回
   
      

> 从状态修改机制和结果返回机制这2点来说，可以认为Future和Promise之间还有一个区别：读写权限不一样——Future只读，Promise可写。具体地说，相对Future，Promise允许用户修改状态和返回结果。


## Future和Promise的示例

下面提供了Future示例和Promise示例，帮助理解上述所讲的二者的不同点：

> - Future示例使用Dart编写
> - Promise示例使用JavaScript编写
> - 在线运行示例代码：[https://repl.it](https://repl.it)（`repl.it`是一个为各种编程语言提供线上编程环境的网站）
> 

- Future示例（by Dart）

  ```dart
  Future<String> sendEcho(text) {
    return new Future.delayed(Duration(seconds: 1), () {
      if (text == null) {
        // the Future becomes completed status after throw(error)
        throw "text can't be null";
      }

      if (text == "") {
        // the Future becomes completed status after return
        return new Future.error("text can't be empty string");
      }

      // the Future becomes completed status after return
      return text;
    });
  }

  echo(text) {
    sendEcho(text).then((value) {
      print("get echo: $value");
    }).catchError((error) {
      print("get error: $error");
    });
  }

  void main() {
    print("echo now ...");
    echo("hello");
    echo("");
    echo(null);
  }
  ```
  
  > 输出结果为：
  >
  > ```
  > echo now ...
  > get echo: hello
  > get error: text can't be empty string
  > get error: text can't be null
  > ```

- Promise示例（by JavaScript）

  ```js
  function sendEcho(text) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (text == null) {
          // the Promise is still pending status after throw(error)
          throw new Error("text can't be null");
          return
        }

        if (text == "") {
          // the Promise becomes rejected status after reject(reason)
          reject("text can't be empty string")
          return
        }

        // the Promise becomes fulfilled status after resolve(value)
        resolve(text)
      }, 1000)
    })
  }

  function echo(text) {
    sendEcho(text).then((value) => {
      console.log("get echo: ", value)
    }).catch(
      (reason) => {
        console.log("get error: ", reason);
      });
  }

  console.log("echo now ...")
  echo("hello")
  echo("")
  echo(null)
  ```
  
  > 输出结果为：
  >
  > ```
  > echo now ...
  > get echo: hello
  > get error: text can't be empty string
  > ```
  
## 参考资料

- [WiKi-《Futures and promises》](https://en.wikipedia.org/wiki/Futures_and_promises)
- [《漫谈并发编程：Future模型（Java、Clojure、Scala多语言角度分析）》](https://cloud.tencent.com/developer/article/1135972)
- [《Futures and Promises》](http://dist-prog-book.com/chapter/2/futures.html)
- [《MDN web docs: Promise》](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [《最浪漫的抽象 Promise & Future》](https://blog.makeex.com/2016/04/30/best-romantic-abstract-promise-and-future/)


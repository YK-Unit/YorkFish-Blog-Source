---
title: Swift5之我对@propertyWrapper的思考二
date: 2022-12-24 12:34:56
categories: ["技术"]
tags: ["2022","iOS", "Swift"]
comments: true
---

## 前言

书接上篇[《Swift5之我对@propertyWrapper的思考一》](https://juejin.cn/post/7147182350147797022)尾部所抛出的一个问题：如何解决`@propertyWrapper`带来的负向影响？

我目前的一个答案是：使用其他方案代替`@propertyWrapper`实现代码复用以及和其类似的语法糖效果。

那么该如何做呢？我的探索答案是：自定义数据类型+字面量协议+“重载强制类型转换操作符”。

下面让我结合上篇中《`@propertyWrapper`方案示例》，对我的探索答案一一展开阐述。

```
 // @propertyWrapper方案示例
 // ======= 实现的 propertyWrapper TwelveOrLess ========
 @propertyWrapper
 struct TwelveOrLess {
     var wrappedValue: Int {
         didSet {
             wrappedValue = min(wrappedValue, 12)
         }
     }
 
     init(wrappedValue: Int) {
         self.wrappedValue = min(wrappedValue, 12)
     }
 }
 
 // ======= 测试用例 ========
 struct NormalRectangle {
     var height: Int
     var width: Int
 }
 
 struct ConstrainedRectangle {
     @TwelveOrLess var height: Int
     @TwelveOrLess var width: Int
 }
 
 func test_TwelveOrLess() {
     let normalRectangle = NormalRectangle(height: 24, width: 24)
     print("(type(of: normalRectangle.height))") // Prints "Int"
 
     let constrainedRectangle = ConstrainedRectangle(height: 24, width: 24)
     print("(type(of: constrainedRectangle.height))") // Prints "TwelveOrLess"
 }
```

## 自定义数据类型

万变不离其宗，实现代码复用的一种方式，就是自定义数据类型，封装相同逻辑。而`@propertyWrapper`的做法也是让开发者按照约定定义一种数据类型，来实现代码复用。下面，就让我们对照例子，先实现一个自定义的数据类型`TwelveOrLess`和对应的测试用例吧：

```
 // ======= 实现的自定义数据类型 TwelveOrLess ========
 struct TwelveOrLess {
     var wrappedValue: Int {
         didSet {
             wrappedValue = min(wrappedValue, 12)
         }
     }
 
     init(wrappedValue: Int) {
         self.wrappedValue = min(wrappedValue, 12)
     }
 }
 
 // ======= 测试用例 ========
 struct ConstrainedRectangle {
     var height: TwelveOrLess
     var width: TwelveOrLess
 }
 
 func test_TwelveOrLess() {
     // 报错：Cannot convert value of type 'Int' to expected argument type 'TwelveOrLess'
     let constrainedRectangle = ConstrainedRectangle(height: 24, width: 24)
     print("(type(of: constrainedRectangle.height))") // Prints "TwelveOrLess"
 }
```

与上篇《`@propertyWrapper`方案示例》对比，`ConstrainedRectangle`中的属性`height`和`width`在编译时和运行时都是明确的数据类型：`TwelveOrLess`——这种显性的、前后一致的表达和表现，能有效帮助让开发者特别是初级开发者避免遇到上篇示例展示的那样疑惑：属性`height`和`width`在运行时类型为何产生了突变。

不过，这时候编译时，我们会遇到报错：`Cannot convert value of type 'Int' to expected argument type 'TwelveOrLess'`。

因为在编译时，属性`height`和`width`是`TwelveOrLess`类型，因此无法接受`Int`类型的字面量值进行实例初始化。而在上篇《`@propertyWrapper`方案示例》中，可以使用`Int`类型的字面量值进行实例初始化，为了对齐《`@propertyWrapper`方案》的开发体验，我们应该怎么做呢？那就是使用字面量协议。

## 字面量协议

> 字面量协议是Swift 提供给自定义数据类型实现和基础数据类型一样的通过字面量进行实例初始化的能力。此处不作过多展开，有兴趣的童鞋可看我的这篇介绍文章：[《Swift的字面量类型（Literal Type）和字面量协议（Literal Protocol）》](https://blog.yorkfish.me/2017/Swift%E7%9A%84%E5%AD%97%E9%9D%A2%E9%87%8F%E7%B1%BB%E5%9E%8B%EF%BC%88Literal%20Type%EF%BC%89%E5%92%8C%E5%AD%97%E9%9D%A2%E9%87%8F%E5%8D%8F%E8%AE%AE%EF%BC%88Literal%20Protocol%EF%BC%89/readme/)

为了解决上述报错以及对齐《`@propertyWrapper`方案》的开发体验，只需要让`TwelveOrLess`实现`ExpressibleByIntegerLiteral`字面量协议即可：

```
 // ======= 实现的自定义数据类型 TwelveOrLess ========
 struct TwelveOrLess {
     var wrappedValue: Int {
         didSet {
             wrappedValue = min(wrappedValue, 12)
         }
     }
 
     init(wrappedValue: Int) {
         self.wrappedValue = min(wrappedValue, 12)
     }
 }
 
 // 实现ExpressibleByIntegerLiteral字面量协议
 extension TwelveOrLess: ExpressibleByIntegerLiteral {
     typealias IntegerLiteralType = Int
 
     public init(integerLiteral value: IntegerLiteralType) {
         self.init(wrappedValue: value)
     }
 }
 
 // ======= 测试用例 ========
 struct ConstrainedRectangle {
     var height: TwelveOrLess
     var width: TwelveOrLess
 }
 
 func test_TwelveOrLess() {
     let constrainedRectangle = ConstrainedRectangle(height: 24, width: 24)
     print("(type(of: constrainedRectangle.height))") // Prints "TwelveOrLess"
 }
```

目前看起来开发体验和《`@propertyWrapper`方案》差不多了，但是还是有短板，那就是赋值操作时，会报错：`Cannot convert value of type 'TwelveOrLess' to specified type 'Int'` 。

```
 func test_TwelveOrLess() {
     // 报错：Cannot convert value of type 'TwelveOrLess' to specified type 'Int'
     var temp_h: Int = constrainedRectangle.height
     print("temp_h: (temp_h)")
 }
```

这时候该怎么解决呢？那就是“重载强制类型转换操作符”，在其赋值给其他类型时可进行强制转换。

## “重载强制类型转换操作符”

然而，很不幸地，目前Swift不支持重载强制类型转换操作符！！！

对此，这个探索方案卡在此处。但是，这里将会通过C++展示这个探索方案，帮助读者一窥全貌：

```
 #include <iostream>
 
 class TwelveOrLess {  
 public:
   int wrappedValue;
   TwelveOrLess() {
   }
   
   // 重载默认赋值运算符 =（相当于Swift的实现“字面量协议”）
   TwelveOrLess& operator = (const int& right) {
     wrappedValue = std::min(12,right);
     return *this;
   }
  
   // 重载强制类型转换操作符（type conversion operator）
   operator int() {
     return wrappedValue;
   }
 };
 
 int main() {
   // 测试字面量初始化实例
   TwelveOrLess t;
   t = 24;
   std::cout << "t: " << t.wrappedValue << std::endl; // Prints "12"
   // 测试转换为基础类型：重载强制类型转换操作符后，赋值给基础类型时，会进行自动强转。
   int i = 24;
   i = t; //等价于 i = t. operator int()
   std::cout << "i: " << i << std::endl; // Prints "12"
 }
```

参照C++的做法，假若Swift支持“重载强制类型转换操作符”，那么，就可以实现`TwelveOrLess`赋值给Int类型变量的开发体验。然而目前，Swift却不支持。那还有没其他解决方案呢？一种解决方案就是实现一个和赋值运算符（`=`）相似自定义操作符。

## 实现自定义操作符

Swift目前是支持开发者实现自定义操作符，因此接着上述思路，我们可给`TwelveOrLess`实现一个和赋值运算符（`=`）相似自定义操作符`<<`，然后通过这个操作符完成赋值操作：

```
 // ======= 实现的自定义数据类型 TwelveOrLess ========
 struct TwelveOrLess {
     var wrappedValue: Int {
         didSet {
             wrappedValue = min(wrappedValue, 12)
         }
     }
 
     init(wrappedValue: Int) {
         self.wrappedValue = min(wrappedValue, 12)
     }
 }
 
 // 实现ExpressibleByIntegerLiteral字面量协议
 extension TwelveOrLess: ExpressibleByIntegerLiteral {
     typealias IntegerLiteralType = Int
 
     public init(integerLiteral value: IntegerLiteralType) {
         self.init(wrappedValue: value)
     }
 }
 
 // 实现自定义操作符 <<
 infix operator <<
 extension TwelveOrLess {
     static func <<(left: inout Int, right: TwelveOrLess) {
         left = right.wrappedValue
     }
 }
 
 // ======= 测试用例 ========
 struct ConstrainedRectangle {
     var height: TwelveOrLess
     var width: TwelveOrLess
 }
 
 func test_TwelveOrLess() {
     let constrainedRectangle = ConstrainedRectangle(height: 24, width: 24)
     print("(type(of: constrainedRectangle.height))") // Prints "TwelveOrLess"
   
     var temp_h: Int = 21
     temp_h << constrainedRectangle.height
     print("temp_h: (temp_h)") // Prints "12"
 }
```

至此，在赋值操作方面算是接近了《`@propertyWrapper`方案》的开发体验。

## 总结

目前探索出来的方案在代码复用、字面量初始化方面都是对齐《`@propertyWrapper`方案》的开发体验，并且保障了开发者的代码感知在编译时和运行时是一致的。但是，由于Swift暂不支持重载强制类型转换操作符这点，导致在赋值操作方面稍差人意。若读者有其他好的想法，欢迎评论交流。


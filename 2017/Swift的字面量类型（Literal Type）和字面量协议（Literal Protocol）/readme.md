---
title: Swift的字面量类型（Literal Type）和字面量协议（Literal Protocol）
date: 2017-10-28 12:34:56
categories: ["技术"]
tags: ["2017", "iOS"]
comments: true
---

## 前言

你自定义了一个数据类型后，是否希望像`var num: Int = 10`这样通过一个字面量初始化一个类型的实例呢？是的话，请看下文详细介绍。

<!-- more -->

## 一、字面量类型（Literal Type）

在介绍字面量类型前，我们先认识下字面量的概念。

**所谓字面量，是指一段能表示特定类型的值（如数值、布尔值、字符串）的源码表达式（it is the source code representation of a fixed value）**。比如下面例子：

``` Swift
let num: Int = 10
let flag: Bool = true
let str: String = "hello"
```

例子中的`10`、`true`、`hello`都是字面量。

那什么是字面量类型呢？

**字面量类型就是支持通过字面量进行实例初始化的数据类型**，如例子中的`Int`、`Bool`、`String`类型。

在Swift中，其的字面量类型有：

- 所有的数值类型: Int、Double、Float以及其的相关类型（如UInt、Int16、Int32等）
- 布尔值类型：Bool
- 字符串类型：String
- 组合类型：Array、Dictionary、Set
- 空类型：Nil

## 二、字面量协议（Literal Protocol）

Swift是如何让上述的数据类型具有字面量初始化的能力呢？

**答案是：实现指定的字面量协议。**

所以，如果我们希望自定义的数据类型也能通过字面量进行初始化，只要实现对应的字面量协议即可。

Swift中的字面量协议主要有以下几个：

- `ExpressibleByNilLiteral` // nil字面量协议
- `ExpressibleByIntegerLiteral` // 整数字面量协议
- `ExpressibleByFloatLiteral` // 浮点数字面量协议
- `ExpressibleByBooleanLiteral` // 布尔值字面量协议
- `ExpressibleByStringLiteral` // 字符串字面量协议
- `ExpressibleByArrayLiteral` // 数组字面量协议
- `ExpressibleByDictionaryLiteral` // 字典字面量协议

其中， `ExpressibleByStringLiteral` 字符串字面量协议相对复杂一点，该协议还依赖于以下2个协议（也就是说，实现`ExpressibleByStringLiteral`时，还需要实现下面2个协议）:

  - `ExpressibleByUnicodeScalarLiteral`
  - `ExpressibleByExtendedGraphemeClusterLiteral`

>在 Swift3.0 之前，上述的字面量协议的名称对应如下：
>- `NilLiteralConvertible`
>- `IntegerLiteralConvertible`
>- `FloatLiteralConvertible`
>- `BooleanLiteralConvertible`
>- `StringLiteralConvertible`
>- `ArrayLiteralConvertible`
>- `DictionaryLiteralConvertible`
>

## 三、字面量协议例子（Literal Protocol Example）

下面将会通过具体例子为大家演示如何通过实现上述的字面量协议。

> 下面的例子均已经上传GitHub，查看下载请点击[LiteralProtocolExample](https://github.com/YK-Unit/Swift-LiteralProtocolExample)

##### 1、定义`Moeny`类型，实现通过整数字面量、浮点数字面量、字符串字面量、布尔值字面量初始化`Money`实例：

``` Swift
//: Playground - noun: a place where people can play

import UIKit
import Foundation

struct Money {
    var value: Double

    init(value: Double) {
        self.value = value
    }
}

// 实现CustomStringConvertible协议，提供description方法
extension Money: CustomStringConvertible {
    public var description: String {
        return "\(value)"
    }
}

// 实现ExpressibleByIntegerLiteral字面量协议
extension Money: ExpressibleByIntegerLiteral {
    typealias IntegerLiteralType = Int

    public init(integerLiteral value: IntegerLiteralType) {
        self.init(value: Double(value))
    }
}

// 实现ExpressibleByFloatLiteral字面量协议
extension Money: ExpressibleByFloatLiteral {
    public init(floatLiteral value: FloatLiteralType) {
        self.init(value: value)
    }
}

// 实现ExpressibleByStringLiteral字面量协议
extension Money: ExpressibleByStringLiteral {

    public init(stringLiteral value: StringLiteralType) {
        if let doubleValue = Double(value) {
            self.init(value: doubleValue)
        } else {
            self.init(value: 0)
        }
    }

    // 实现ExpressibleByExtendedGraphemeClusterLiteral字面量协议
    public init(extendedGraphemeClusterLiteral value: StringLiteralType) {
        if let doubleValue = Double(value) {
            self.init(value: doubleValue)
        } else {
            self.init(value: 0)
        }
    }

    // 实现ExpressibleByUnicodeScalarLiteral字面量协议
    public init(unicodeScalarLiteral value: StringLiteralType) {
        if let doubleValue = Double(value) {
            self.init(value: doubleValue)
        } else {
            self.init(value: 0)
        }
    }
}

// 实现ExpressibleByBooleanLiteral字面量协议
extension Money: ExpressibleByBooleanLiteral {
    public init(booleanLiteral value: BooleanLiteralType) {
        let doubleValue: Double = value ? 1.0 : 0.0
        self.init(value: doubleValue)
    }
}

// 通过整数字面量初始化
let intMoney: Money = 10

// 通过浮点数字面量初始化
let floatMoney: Money = 10.1

// 通过字符串字面量初始化
let strMoney: Money = "10.2"

// 通过布尔值初始化
let boolMoney: Money = true

```
##### 2、定义`Book`类型，实现通过字典字面量、数组字面量、nil字面量初始化`Book`实例:

``` Swift
//: Playground - noun: a place where people can play

import Foundation

struct Book {

    public var id: Int
    public var name: String

    init(id: Int, name: String = "unnamed") {
        self.id = id
        self.name = name
    }
}

// 实现CustomStringConvertible协议，提供description方法
extension Book: CustomStringConvertible {
    public var description: String {
        return "id:\(id)\nname:《\(name)》"
    }
}

// 实现ExpressibleByDictionaryLiteral字面量协议
extension Book: ExpressibleByDictionaryLiteral {
    typealias Key = String
    typealias Value = Any

    public init(dictionaryLiteral elements: (Key, Value)...) {
        var dictionary = [Key: Value](minimumCapacity: elements.count)
        for (k, v) in elements {
            dictionary[k] = v
        }

        let id = (dictionary["id"] as? Int) ?? 0
        let name = (dictionary["name"] as? String) ?? "unnamed"

        self.init(id: id, name: name)
    }
}

// 实现ExpressibleByArrayLiteral字面量协议
extension Book: ExpressibleByArrayLiteral {
    typealias ArrayLiteralElement = Any

    public init(arrayLiteral elements: ArrayLiteralElement...) {
        var id: Int = 0
        if let eId = elements.first as? Int {
            id = eId
        }

        var name = "unnamed"
        if let eName = elements[1] as? String {
            name = eName
        }

        self.init(id: id, name: name)
    }
}

// 实现ExpressibleByNilLiteral字面量协议
extension Book: ExpressibleByNilLiteral {
    public init(nilLiteral: ()) {
        self.init()
    }
}

// 通过字典字面量初始化
let dictBook: Book = ["id": 100, "name": "Love is Magic"]
print("\(dictBook)\n")

// 通过数组字面量初始化
let arrayBook: Book = [101, "World is word"]
print("\(arrayBook)\n")

// 通过nil字面量初始化
let nilBook: Book = nil
print("\(nilBook)\n")

```

## 四、关于 'not expressible by any literal
enum' Error

当你使用自定义数据类型定义枚举时，可能会遇到以下类似错误：

>raw type 'XX_TYPE' is not expressible by any literal
enum XX_ENUM: XX_TYPE

这是说你的自定义数据类型没有实现字面量协议。然而需要注意的是，**enum目前支持的字面量协议是有限制的，其目前只支持以下几个字面量协议：**

- ExpressibleByIntegerLiteral
- ExpressibleByFloatLiteral
- ExpressibleByStringLiteral

也就是说，若你的自定义数据类型实现的字面量协议没有包含上面中的一个，就会得到此种错误。具体示例如下：

``` Swift
//: Playground - noun: a place where people can play

import Foundation

struct StockType {
    var number: Int
}

// 实现CustomStringConvertible协议，提供description方法
extension StockType: CustomStringConvertible {
    public var description: String {
        return "Stock Number:\(number)"
    }
}

// 实现Equatable协议，提供==方法
extension StockType: Equatable {
    public static func ==(lhs: StockType, rhs: StockType) -> Bool {
        return lhs.number == rhs.number
    }
}

// 实现ExpressibleByDictionaryLiteral字面量协议
extension StockType: ExpressibleByDictionaryLiteral {
    typealias Key = String
    typealias Value = Any

    public init(dictionaryLiteral elements: (Key, Value)...) {
        var dictionary = [Key: Value](minimumCapacity: elements.count)
        for (k, v) in elements {
            dictionary[k] = v
        }

        let number = (dictionary["number"] as? Int) ?? 0

        self.init(number: number)
    }
}

// 实现ExpressibleByIntegerLiteral字面量协议
extension StockType: ExpressibleByIntegerLiteral {
    public init(integerLiteral value: IntegerLiteralType) {
        self.init(number: value)
    }
}

/*
 若StockType没有实现 ExpressibleByIntegerLiteral、ExpressibleByFloatLiteral、ExpressibleByStringLiteral中的一个，会报错误：error: raw type 'StockType' is not expressible by any literal
 */
// 你可以尝试去掉ExpressibleByIntegerLiteral的实现，看看编译器报的错误
enum Stock: StockType {
    case apple = 1001
    case google = 1002
}

let appleStock = Stock.apple.rawValue
print("\(appleStock)")

```

上述例子中，定义了`StockType`数据类型和`Stock`枚举类型。若`StockType`去掉`ExpressibleByIntegerLiteral`字面量的协议的实现，将会获得上述的编译错误。

## 资料参考
- [Initialization with Literals](https://developer.apple.com/documentation/swift/initialization_with_literals)
- [Swift: Raw{Not}Representable enum](http://blog.krzyzanowskim.com/2015/03/12/swift-raw-not-representable-enum/)
- [Swift - Literals](https://www.tutorialspoint.com/swift/swift_literals.htm)
- [Swift Literal Convertibles](http://nshipster.com/swift-literal-convertible/)
- [字面量转换](http://swifter.tips/literal/)


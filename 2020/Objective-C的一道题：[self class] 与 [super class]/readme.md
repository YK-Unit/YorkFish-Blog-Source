---
title: Objective-C的一道题：[self class] 与 [super class]
date: 2020-05-12 10:10:01
categories: ["iOS", "基础理论"]
tags: ["2020"]
comments: true
---



## 前言

这是一道从此篇博客[《神经病院 Objective-C Runtime 入院第一天—— isa 和 Class》](https://halfrost.com/objc_runtime_isa_class/)看到的题目：

> 问：下面代码输出什么?
> 
> ```objective-c
> @implementation Son : Father
> - (id)init
> {
>     self = [super init];
>     if (self)
>     {
>         NSLog(@"%@", NSStringFromClass([self class]));
>         NSLog(@"%@", NSStringFromClass([super class]));
>     }
> return self;
> }
> @end
> ```

答案是： 输出的都是`Son`。

对此，博客作者给出的理由是：

> self和super的区别：
>
> self是类的一个隐藏参数，每个方法的实现的第一个参数即为self。
>
> super并不是隐藏参数，它实际上只是一个”编译器标示符”，它负责告诉编译器，当调用方法时，去调用父类的方法，而不是本类中的方法。
>
> 在调用[super class]的时候，runtime会去调用objc_msgSendSuper方法，而不是objc_msgSend
>
> ```objective-c
> OBJC_EXPORT void objc_msgSendSuper(void /* struct objc_super *super, SEL op, ... */ )
> 
> 
> /// Specifies the superclass of an instance. 
> struct objc_super {
>     /// Specifies an instance of a class.
>     __unsafe_unretained id receiver;
> 
>     /// Specifies the particular superclass of the instance to message. 
> #if !defined(__cplusplus)  &&  !__OBJC2__
>     /* For compatibility with old objc-runtime.h header */
>     __unsafe_unretained Class class;
> #else
>     __unsafe_unretained Class super_class;
> #endif
>     /* super_class is the first class to search */
> };
> ```
>
> 在objc_msgSendSuper方法中，第一个参数是一个objc_super的结构体，这个结构体里面有两个变量，一个是接收消息的receiver，一个是
> 当前类的父类super_class。
>
> 入院考试第一题错误的原因就在这里，误认为[super class]是调用的[super_class class]。
>
> objc_msgSendSuper的工作原理应该是这样的:
> 从objc_super结构体指向的superClass父类的方法列表开始查找selector，找到后以objc->receiver去调用父类的这个selector。注意，最后的调用者是objc->receiver，而不是super_class！
>
> 那么objc_msgSendSuper最后就转变成
>
> ```objectivec
> // 注意这里是从父类开始msgSend，而不是从本类开始，谢谢@Josscii 和他同事共同指点出此处描述的不妥。
> objc_msgSend(objc_super->receiver, @selector(class))
> 
> /// Specifies an instance of a class.  这是类的一个实例
>     __unsafe_unretained id receiver;   
> 
> 
> // 由于是实例调用，所以是减号方法
> - (Class)class {
>     return object_getClass(self);
> }
> ```
>
> 由于找到了父类NSObject里面的class方法的IMP，又因为传入的入参objc_super->receiver = self。self就是son，调用class，所以父类的方法class执行IMP之后，输出还是son，最后输出两个都一样，都是输出son。

其实，上述的解答只答对了一大半，还有一小半没正确。没正确的部分主要在于对`objc_msgSend`和`objc_msgSendSuper`的工作原理理解错误，以及对方法实现的函数原型的忽略。

下面将会对此进行重新解答。

<!-- more -->

## 新解

Objective-C 是一门基于消息转发机制的动态编程语言，所有的方法调用都会转换为消息发送。所以，`[self class]`和`[super class]`会被编译器分别转换为`objc_msgSend(self, @selector(class))`和`objc_msgSendSuper(objc_super, @selector(class))`。

> 具体地是转换为以下的C++ 代码：
>
> ```C++
> // [self class]
> ((Class (*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("class"));
> 
> // [super class]
> ((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("Son"))}, sel_registerName("class"));
> ```
>
> > 执行`clang -rewrite-objc xx.m -o xx.cpp`即可转换。

`objc_msgSend`会根据`SEL`从当前对象开始查找对应的方法实现，查找到方法实现后，就会直接跳转到方法实现。

`objc_msgSendSuper`会根据`SEL`从当前对象的父类开始查找对应的方法实现，查找到方法实现后，就会直接跳转到方法实现。

> `objc_msgSend`的具体工作原理请看下面代码：
>
> ```asm
> /********************************************************************
>  *
>  * id objc_msgSend(id self, SEL _cmd, ...);
>  * IMP objc_msgLookup(id self, SEL _cmd, ...);
>  * 
>  * objc_msgLookup ABI:
>  * IMP returned in r12
>  * Forwarding returned in Z flag
>  * r9 reserved for our use but not used
>  *
>  ********************************************************************/
> 
> 	ENTRY _objc_msgSend
> 	
> 	cbz	r0, LNilReceiver_f
> 
> 	ldr	r9, [r0]		// r9 = self->isa
> 	GetClassFromIsa			// r9 = class
> 	CacheLookup NORMAL, _objc_msgSend
> 	// cache hit, IMP in r12, eq already set for nonstret forwarding
> 	bx	r12			// call imp
> 
> 	CacheLookup2 NORMAL, _objc_msgSend
> 	// cache miss
> 	ldr	r9, [r0]		// r9 = self->isa
> 	GetClassFromIsa			// r9 = class
> 	b	__objc_msgSend_uncached
> 
> ```
>
> `objc_msgSendSuper`的具体工作原理请看下面代码：
>
> ```asm
> /********************************************************************
>  * id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
>  *
>  * struct objc_super {
>  *     id receiver;
>  *     Class cls;	// the class to search
>  * }
>  ********************************************************************/
> 
> 	ENTRY _objc_msgSendSuper
> 	
> 	ldr	r9, [r0, #CLASS]	// r9 = struct super->class
> 	CacheLookup NORMAL, _objc_msgSendSuper
> 	// cache hit, IMP in r12, eq already set for nonstret forwarding
> 	ldr	r0, [r0, #RECEIVER]	// load real receiver
> 	bx	r12			// call imp
> 
> 	CacheLookup2 NORMAL, _objc_msgSendSuper
> 	// cache miss
> 	ldr	r9, [r0, #CLASS]	// r9 = struct super->class
> 	ldr	r0, [r0, #RECEIVER]	// load real receiver
> 	b	__objc_msgSend_uncached
> 	
> 	END_ENTRY _objc_msgSendSuper
> ```
>
> 需要注意的是，Objective-C根据方法的调用者和方法的返回结果，提供了不同的消息发送函数，具体如下：
>
> - 调用者是对象的父类（an object’s superclass, using the super keyword），返回结果类型为结构体数据类型（ data structures return types）的消息使用`objc_msgSendSuper`发送
> - 调用者是对象的父类（an object’s superclass, using the super keyword）的消息使用`objc_msgSendSuper`发送
> - 调用者是对象的（an object），返回结果类型为结构体数据类型（ data structures return types）的消息使用`objc_msgSend_stret`发送
> - 调用者是对象的（an object），返回结果类型为浮点数类型（ some float return types）的消息使用`objc_msgSend_fpret`发送
> - 调用者是对象的（an object），返回结果为复杂浮点数类型（some complex float return types）的消息使用`objc_msgSend_fp2ret`发送
> - 剩余消息使用`objc_msgSend`发送
>
> 其中，关于`objc_msgSend_fpret`和`objc_msgSend_fp2ret`的更多细节，请看下面引自源代码中的注释：
>
> >  Floating-point-returning Messaging Primitives
> >
> >  Use these functions to call methods that return floating-point values 
> >  on the stack. 
> >  Consult your local function call ABI documentation for details.
> >
> >  arm:    objc_msgSend_fpret not used
> >  i386:   objc_msgSend_fpret used for `float`, `double`, `long double`.
> >  x86-64: objc_msgSend_fpret used for `long double`.
> >
> >  arm:    objc_msgSend_fp2ret not used
> >  i386:   objc_msgSend_fp2ret not used
> >  x86-64: objc_msgSend_fp2ret used for `_Complex long double`.
> >
> >  These functions must be cast to an appropriate function pointer type 
> >  before being called. 

由于`class`方法在`NSObject`处，最终`objc_msgSend(self, @selector(class))`和`objc_msgSendSuper(objc_super, @selector(class))`查找到是同一个方法实现。

在Objective-C 中，方法实现的函数原型如下：

```c
/// A pointer to the function of a method implementation. 
#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ ); 
#else
typedef id _Nullable (*IMP)(id _Nonnull, SEL _Nonnull, ...); 
#endif
```

`IMP`为方法实现的函数原型，其有2个固定的参数：`id`和`SEL`，其中`id`是运行时接收消息（即进行方法调用）的对象实例。

`objc_msgSend(self, @selector(class))`和`objc_msgSendSuper(objc_super, @selector(class))`传递给`class`这个方法的`IMP`的参数`id`都是同一个对象实例，所以最终二者的输出是相同的。

## 参考资料

- [objc-runtime源码](https://github.com/0xxd0/objc4)


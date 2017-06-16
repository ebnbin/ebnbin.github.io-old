---
layout: post
date: 2017-06-16 16:24:47 +0800
title: 翻译：Kotlin 变量, 使用 Lazy 还是 Late?
tags:
    - Kotlin
    - Android
---

> Original（原文）：[Kotlin variable, to be Lazy, or to be Late?](https://medium.com/@elye.project/kotlin-variable-to-be-lazy-or-to-be-late-ab865c682d61)

今天我们介绍 Kotlin 中的两个新特性，基于一个老的概念——[lazy initialization](https://en.wikipedia.org/wiki/Lazy_initialization)（懒初始化），即：推迟一个变量的初始化时机。这是个十分方便的特性，因为我们通常会遇到这样的情况，一个变量直到使用时才需要被初始化，或者仅仅是它的初始化依赖于某些无法立即获得的上下文。

让我先来介绍两种语法

#### [Initialization by Lazy](https://en.wikipedia.org/wiki/Lazy_initialization)（懒初始化）

``` Kotlin
val myUtil by lazy {
     MyUtil(parameter1, parameter2)
}
```

上面这段代码创建了一个 *`MyUtil`* 类型的对象。但这只会发生在第一次使用 *`myUtil`* 的时候。

#### [Late Initialization](https://kotlinlang.org/docs/reference/properties.html#late-initialized-properties)（延迟初始化）

``` Kotlin
lateinit var myUtil: MyUtil
```

之后在某个函数中

``` Kotlin
myUtil = MyUtil(parameter1, parameter2)
```

大致来说，这段代码将变量的初始化与定义分离开了。

## 为什么有两种方式？使用场景如何？

尽管两种方式有着相同的理念，本质上却有着很大的差异。简单地看下变量的类型，一个是 ***`val`***（值不可变），另一个是 ***`var`***（值可变）。这已经很能说明问题了。下面是一些典型的使用场景

#### 单例变量

有时，我们需要的只是变量的单个实例，全局共享这个变量

``` java
MyUtil getMyUtil() {
    if (myUtil == null) myUtil = new MyUtil(param1, param2);
    return myUtil;
}
```

这在单例模式中是非常常见的，我们可以确保所有地方访问的都是同一个实例。同时也能保证这个对象只在我们需要时被创建。在这种情况下，*Lazy Initialization（懒初始化模式）* 将很有用。

#### 初始化一个不可为空的变量

在 Kotlin 中，我们需要预先定义好一个变量是否可为空。这使得编译器能在编译时识别出潜在的 `null` 对象，以避免 *NPE （空指针异常）*。

因此，对于一个 *不可为空* 的成员变量，当类被创建时，它必须要被赋予一个非空的值。如果在类被创建时，初始化这个成员变量的所有依赖对象都已经准备好，那当然就不是问题了。不幸的是，有些情况下，这些依赖对象（例如上述例子中的 *`param1`* 和 *`param2`*）在运行时才可用。这么一来，我们就陷入困境了，因为我们没法让这个变量在初始化时先赋值为 *`null`*（Kotlin 不允许我们这么做），而我们又需要等到相关依赖对象都准备好了才能给它初始化。

基于这种情况，***lateinit（延迟初始化模式）*** 给我们带来了便利，Kotlin 以优雅的方式允许一个成员变量在定义时是未初始化的，之后当一切准备就绪了再执行它的初始化。

#### 使用依赖注入的变量

如果你在 Kotlin 中使用了依赖注入框架（例如 `Dagger2`），那些预先声明的变量也无法成功初始化。在这种情况下，必须使用 ***`lateinit`*** 关键字确保变量在稍后将被初始化。

``` Kotlin
@Inject
lateinit var myUtil: MyUtil
```

实际上，***`lateinit`*** 关键字就是因为这个原因而在 Kotlin 中明确地引入的。

## 何时用？用哪个？

上述例子仅是一小部分使用情境。事实上只要是不可为空的变量，很多情况下都是适用的。有一些简单的规则来帮助你决定该用哪一个模式

1. 如果是值可修改的变量（即在之后的使用中可能被重新赋值），使用 ***lateInit*** 模式

2. 如果变量的初始化取决于外部对象（例如需要一些外部变量参与初始化），使用 ***lateInit*** 模式。这种情况下，*lazy* 模式也可行但并不直接适用。

3. 如果变量仅仅初始化一次并且全局共享，且更多的是内部使用（依赖于类内部的变量），请使用 *lazy* 模式。从实现的角度来看，***lateinit*** 模式仍然可用，但 *lazy* 模式更有利于封装你的初始化代码。

综上所述，不考虑对变量值是否可变的控制，***lateinit*** 模式是 *lazy* 模式的超集，你可以在任何使用 *lazy* 模式的地方用 ***lateinit*** 模式替代，反之则不然。但在可能的情况下，请尽量使用 *lazy* 模式，因为 ***lateinit*** 模式在函数中暴露了太多的逻辑代码，使得代码更加混乱，相比而言，*lazy* 模式更好地封装了细节，且更安全。

总结而言，对于变量的初始化，优先选择 *lazy* 模式，其次再考虑 *late* 模式……实在不行，选择最原始的方法手动实现。尽量做个懒人吧，[比尔盖茨喜欢他们](http://www.goodreads.com/quotes/568877-i-choose-a-lazy-person-to-do-a-hard-job)。

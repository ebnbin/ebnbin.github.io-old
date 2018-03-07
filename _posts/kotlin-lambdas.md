---
layout: post
date: 2018-03-07 17:59:27 +0800
title: Kotlin 高阶函数和 lambda 表达式
tags:
    - Kotlin
---

``` java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("clicked");
    }
});
```

以上这段代码在 Java 8 中可以这么写：

``` java
button.setOnClickListener(v -> System.out.println("clicked"));
```

在 Kotlin 中可以这么写：

``` kotlin
button.setOnClickListener { println("clicked") }
```

如何理解这段代码，一步一步来简化。

##### 1. 匿名内部类

要创建一个继承自某类型的匿名类的对象，写法如下（这段代码相当于 Java 8 之前的匿名内部类写法）：

``` kotlin
button.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View?) {
        println("clicked")
    }
})
```

##### 2. 使用 lambda 表达式

约定如下：

* lambda 表达式总是括在花括号中；
* 其参数（如果有的话）在 -> 之前声明；
* 函数体（如果存在的话）在 -> 后面。

``` kotlin
button.setOnClickListener({ v: View? -> println("clicked") })
```

##### 3. 省略参数类型
（参数类型可以省略）

省略类型，形式参数 v 没有使用时可以简写为 `_`：

``` kotlin
button.setOnClickListener({ _ -> println("clicked") })
```

##### 4. 省略参数

如果函数字面值只有一个参数， 那么它的声明可以省略（连同 ->），其名称是 it：

``` kotlin
button.setOnClickListener({ println("clicked") })
```

##### 5. 将花括号移出圆括号

在 Kotlin 中有一个约定，如果函数的最后一个参数是一个函数，并且你传递一个 lambda 表达式作为相应的参数，你可以在圆括号之外指定它。如果 lambda 是该调用的唯一参数，则调用中的圆括号可以完全省略：

``` kotlin
button.setOnClickListener { println("clicked") }
```

一个 lambda 表达式或匿名函数是一个“函数字面值”，即一个未声明的函数，但立即做为表达式传递。接受一个函数作为参数的函数是一个函数。对于接受另一个函数作为参数的函数，我们必须为该参数指定函数类型。

`setOnClickListener` 的函数类型为 

比如：

``` kotlin
fun captureScreenshot(view: View, viewToBitmap: (View) -> Bitmap) {
    val header = createHeader()
    val content = viewToBitmap(view)
    val footer = createFooter()
    saveBitmap(header, content, footer)
}
```

其中 `(View) -> Bitmap` 就是参数 `viewToBitmap` 的函数类型，它接受一个 `View` 返回一个 `Bitmap`，相当于：

什么是 lambda？简单的说就是匿名函数，它是一个“函数字面值”，即一个未声明的函数，但立即做为表达式传递。比如：

``` kotlin
val sum = { x: Int, y: Int -> x + y }

// 使用时：
val result = sum(1, 2)
// 或者:
val result = sum.invoke(1, 2)
```

它相当于：

``` kotlin
fun sum(x: Int, y: Int): Int {
    return x + y
}

// 使用时：
val result = sum(1, 2)
// 或者：
val result = ::sum.invoke(1, 2)
```

标签

Lambda 表达式的完整语法形式，即函数类型的字面值如下：














































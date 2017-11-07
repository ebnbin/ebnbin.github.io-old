---
layout: post
date: 2017-11-07 17:05:28 +0800
title: Kotlin 标识符
tags:
    - Kotlin
---

简单地聊一聊 Kotlin 标识符。

## Java 允许而 Kotlin 不允许的

* `_`、`__`、`___` 等只由下划线组成的标识符在 Kotlin 中是不允许的。但是复合在其他字符中使用是可以的。下划线在 Kotlin 中的使用有几个场景：

> 1. Lambda 表达式中简写参数。

``` kotlin
// 第一个参数是 View, 第二个参数是 MotionEvent. 但他们在函数中都不需要 (没引用), 所以可以使用 _.
view.setOnTouchListener { _, _ -> 
    // Do nothing.

    true // 这个是返回值, 在 Lambda 表达式中不需要也不能够写 return.
}
```

> 2. 提高数字常量可读性。

``` kotlin
val num = 1_000_000
val hex = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010

// PS: Kotlin 不支持八进制数字, Java 中 0 开头的数字是八进制数字.
```

> 3. 数据类和解构声明中省略不需要的属性声明。

``` kotlin
data class Person(var name: String, var age: Int)

val person = Person("Pichai", 45)
val (name, _) = person
println(name) // Pichai
```

* `$` 这个字符在 Kotlin 中不能用在标识符中，它在 Kotlin 中在字符串模版中使用。例如 `"My name is $name!"`。

* Kotlin 中的关键字或修饰符（而 Java 中不是关键字的），例如 `in`、`when`，在 Kotlin 中使用 **\`in\`**, **\`when\`**。其实 Kotlin 中所有标识符都可以在前后添加 **\`** 来使用，就是为了在任何 Koltin 本身不允许作为标识符的情况下做兼容，比如 **\`_\`** 相当于 Java 中的 `_`。当使用的标识符在当前语境下不产生歧义时，关键字或修饰符也可以作为标识符，比如 `data` 在 Kotlin 中是可以用来定义数据类（`data class`）的修饰符，但是 `var data = 0` 这么写是允许的。

## Kotlin 允许而 Java 不允许的

毕竟 Java 是老大，Kotlin 要向 Java 兼容而不是反过来，所以当 Kotlin 中可用而 Java 中不允许的标识符出现时（例如 Java 中的关键字在 Kotlin 中不是关键字的），那么 Kotlin 就要自己通过添加前缀修饰符的方式在字节码层面做兼容了。

## 中文标识符

这在 Kotlin 和 Java 中都允许但 = =。。。 **永远别用！！**

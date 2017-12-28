---
layout: post
date: 2017-12-13 11:34:23 +0800
title: Kotlin 函数
tags:
    - Kotlin
---

# 函数默认参数和命名参数

* 默认参数扩展函数调用方式。

``` kotlin
fun setPadding(
        left: Int = this.left,
        top: Int = this.top,
        right: Int = this.right,
        bottom: Int = this.bottom) {}

setPadding()                                         // 全部使用默认值.
setPadding(1)                                        // 设置 left.
setPadding(1, 2, 3, 4)                               // 设置全部.
setPadding(top = 2)                                  // 设置 top.
//setPadding(left = 1, 2, 3, 4)                      // 编译错误.
setPadding(1, 2, bottom = 4, right = 3)              // 默认参数顺序可以改变.
setPadding(left = 1, top = 2, right = 3, bottom = 4) // 命名参数, 增加可读性.
```

* 默认参数可以使用之前的参数作为默认值。

``` kotlin
fun setMargin(
        left: Int = 0,
        top: Int = 0,
        right: Int = left,
        bottom: Int = top) {}

setMargin()                    // 0, 0, 0, 0
setMargin(1)                   // 1, 0, 1, 0
setMargin(1, 2)                // 1, 2, 1, 2
setMargin(right = 3)           // 0, 0, 3, 0
setMargin(top = 2, bottom = 4) // 0, 2, 0, 4
```

* 默认参数减少方法重载。

``` kotlin
class CustomView(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0, defStyleRes: Int = 0)
```

* 默认参数在无默认参数之前。

``` kotlin
fun circle(x: Float = 0, y: Float = 0, radius: Float) {}

circle(radius = 1f) // 如果要 x, y 使用默认参数, 必须通过命名参数指定 radius.
```

* Lambda 表达式作为最后一个参数。

``` kotlin
fun post(delayMillis: Long = 0L, runnable: () -> Unit) {}

post {
    // Do something.
}
post(1000L) {
    // 1 秒后 do something.
}
post(runnable = {
    // 如果在括号内传递 lambda 表达式参数, 想要使用 delayMillis 默认参数, 仍然必须使用命名参数.
})
```

# 可变数量参数

* 与 Java 的 ... 可变参数类似。

``` kotlin
fun sum(vararg numbers: Float): Float {
    var sum = 0f
    numbers.forEach { sum += it }
    return sum
}

sum(1f, 2f, 3f, 4f)
```

* 可变数量参数作为参数传递。

``` kotlin
fun average(vararg numbers: Float): Float {
//    return sum(numbers) / numbers.size // 编译错误：Type mismatch. Required: Float. Found: FloatArray.
    return sum(*numbers) / numbers.size
}

val floatArray = floatArrayOf(3f, 4f, 5f)
average(1f, 2f, *floatArray, 6f)
```

# 扩展函数

> 能够扩展一个类的新功能而无需继承该类或使用像装饰者这样的任何类型的设计模式。

``` kotlin
fun Date.toCalendar(): Calendar {
    val calendar = Calendar.getInstance()
    calendar.time = this
    return calendar
}

val calendar = date.toCalendar()
```

* 扩展函数不会覆盖成员函数。

``` kotlin
// Warning: Extension is shadowed by a member: public open fun toString(): String
fun Date.toString(): String {
    return "我就不告诉你时间"
}

date.toString() // 还是会输出时间的.

```

* 但是方法重载是可以的。

``` kotlin
fun Date.toString(pattern: String): String {
    val dateFormat = SimpleDateFormat(pattern, Locale.getDefault())
    return dateFormat.format(this)
}

date.toString("yyyy-MM-dd") // 2017-12-13
```

* 可空接受者。
``` kotlin
fun String?.isEmpty(): Boolean {
//    return this == null || this.isEmpty() // 这么写会出现一个小递归。。。
    return this == null || this.length == 0
}

fun foo(str: String?) {
    str.isEmpty()
}
```

* 中缀表示法进一步简化.

``` kotlin
infix fun Int.pow(e: Int): Int {
    return Math.pow(this.toDouble(), e.toDouble()).toInt()
}

2.pow(5) // 不加 infix 调用.
2 pow 5 // 加 infix 简化调用. 32
```

# 局部函数

``` kotlin
fun processor(type: Char) {
    // 需要被复用且仅被 processor 方法复用.
    fun coreOperation() {
        println("Core operation.")
    }

    when (type) {
        'A' -> {
            println("Pre A")
            coreOperation()
            println("Post A")
        }
        'B' -> {
            println("Pre B")
            coreOperation()
            println("Post B")
        }
    }
}
```

# 方法赋予属性

``` kotlin
val action = fun() {
    println("property")
}

val action2 = fun() {
    println("property 2")
}

fun action2() {
    println("function")
}

action()         // property
action2.invoke() // property 2
action2()        // function
```

# 传递成员 function 参数

``` kotlin
fun printAsterisk() {
    println("****************************************************************")
}
fun printPound() {
    println("################################################################")
}
fun printHyphe() {
    println("----------------------------------------------------------------")
}

fun log(message: String, printSeparatorAction: () -> Unit) {
    printSeparatorAction()
    println(message)
    printSeparatorAction()
}

log("hello", ::printAsterisk)
```

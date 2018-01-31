---
layout: post
date: 2017-01-31 23:57:57 +0800
title: Kotlin 操作符重载
tags:
    - Kotlin
---

在 Kotlin 中，我们可以用 **约定的操作符**，代替 **调用代码中以特定的命名定义的函数**，来实现 **与之对应的操作**。例如在类中定义了一个名为 `plus` 的特殊方法，就可以使用加法运算符 `+` 代替 `plus()` 的方法调用。由于你无法修改已有的接口定义，因此一般可以通过 **扩展函数** 来为现有的类增添新的 **约定方法**，从而使得 **操作符重载** 这一语法糖适应任何现有的 Java 类。

## 算术运算符

我们就从最简单直接的例子 `+` 这一类算术运算符开始。

``` kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point) = Point(x + other.x, y + other.y)
    operator fun plus(value: Int) = "toString: ${Point(x + value, y + value)}"
}

fun main(args: Array<String>) {
    val p1 = Point(1, 2)
    val p2 = Point(3, 4)
    println(p1 + p2)
    println(p1 + 3)
}

/*
Point(x=4, y=6)
toString: Point(x=4, y=5)
*/
```

* `operator` 修饰符是必须的，否则 `plus` 只是一个普通方法，不能通过 `+` 调用。

* `plus` 方法的参数类型是任意的，因此可以方法重载，但是参数数量只能是 1，因为 `+` 是一个二元操作符。`plus` 方法的返回值类型也是任意的。

* 如果出现多个方法签名相同的 operator 扩展方法，根据 import 决定使用哪个一，例如：

``` kotlin
// 第一个文件：
package package0

operator fun Point.times(value: Int) = Point(x * value, y * value)

// 第二个文件：
package package1

operator fun Point.times(value: Int) = Unit // Do nothing.

// 使用第一个扩展操作符：
import package0.times

val newPoint = Point(1, 2) * 3
```

Kotlin 为一些基本类型预定义了一些操作符方法，我们平时常写的基本数据计算也可以翻译成调用这些操作符方法，比如 `(2 + 3) * 4` 可以翻译成 `2.plus(3).times(4)`，`2 + 3 * 4` 可以翻译成 `2.plus(3.times(4))`。根据扩展函数的语法，扩展函数无法覆盖与类已有的方法签名相同的方法，因此，不必担心随随便便给 `Int` 自定义一个 `plus` 扩展方法就能让 `1 + 1` 变得不等于 `2`。

所有可重载的算术运算符有：

| 表达式 | 翻译为 |
| :---- | :---- |
| a + b | a.plus(b) |
| a - b | a.minus(b) |
| a * b | a.times(b) |
| a / b | a.div(b) |
| a % b | a.rem(b)、 ~~a.mod(b)~~ （在 Kotlin 1.1 中被弃用） |
| a..b  | a.rangeTo(b) |

它们的优先级与普通的数字类型运算符优先级相同。其中 `rangeTo` 会在下面说明。

## 广义赋值操作符

| 表达式 | 翻译为 |
| :---- | :---- |
| a += b | a.plusAssign(b) |
| a -= b | a.minusAssign(b) |
| a *= b | a.timesAssign(b) |
| a /= b | a.divAssign(b) |
| a %= b | a.remAssign(b)、 ~~a.modAssign(b)~~ （在 Kotlin 1.1 中被弃用） |

对于以上广义赋值操作符：

* 如果对应的二元算术运算符函数也 **可用** ，则报错。`plus` 对应 `plusAssign`。`minus`、`times` 等也类似。

* 返回值类型必须为 `Unit`。

* 如果执行 `a += b` 时 `plusAssign` 不存在，会尝试生成 `a = a + b`，其中的 `a + b` 使用的就是 `plus` 操作符方法，相当于调用 `a = a.plus(b)`。并且此时会要求 `a + b` 的 `plus` 方法的返回值类型必须与 `a` 类型一致（如果单独使用 `a + b` 不做此要求）。

``` kotlin
data class Size(var width: Int = 0, var height: Int = 0) {
    operator fun plus(other: Size): Size {
        return Size(width + other.width, height + other.height)
    }

    operator fun plusAssign(other: Size) {
        width += other.width
        height += other.height
    }
}

fun main(args: Array<String>) {
//    var s1 = Size(1, 2) // 如果这么写，执行 += 时会报错.
    val s1 = Size(1, 2)
    val s2 = Size(3, 4)
    s1 += s2
}
```

我们使用这个例子来理解：为什么使用 `var` 定义的 `s1` 会导致 `+=` 报错呢？因为理论上，执行 `+=` 时，既可以调用 `s1 = s1 + s2`，也就是 `s1 = s1.plus(s2)`，又可以调用 `s1.plusAssign(s2)`，都符合操作符重载约定，这样就会产生歧义，而如果使用 `val` 定义 `s1`，则只可能执行 `s1.plusAssign(s2)`，因为 `s1` 不可被重新赋值，因此 `s1 = s1 + s2` 这样的语法是出错的，永远不能能调用，那么调用 `s1 += s2` 就不会产生歧义了。

既然编译器会帮我把 `a += b` 解释成 `a = a + b`，那是不是意味着我只需要 `plus` 永远不需要 `plusAssign` 了呢？比较好的实践方式是：

* `+` (plus) 始终返回一个新的对象

* `+=` (plusAssign) 用于内容可变的类型，修改自身的内容。

Kotlin 标准库中就是这么实现的：

``` kotlin
fun main(args: Array<String>) {
    val list = arrayListOf(1, 2)
    list += 3              // 添加元素到自身集合, 没有新的对象被创建, 调用的是 add 方法.
    val newList = list + 4 // 创建一个新的 ArrayList, 添加自身元素和新元素并返回新的 ArrayList.
}
```

## rangeTo

`rangeTo` 用于创建一个区间。例如 `1..10` 也就是 `1.rangeTo(10)` 代表了从 `1` 到 `10` 这 10 个数字，`Int.rangeTo` 方法返回一个 `IntRange` 对象，`IntRange` 类定义如下：

``` kotlin
/**
 * A range of values of type `Int`.
 */
public class IntRange(start: Int, endInclusive: Int) : IntProgression(start, endInclusive, 1), ClosedRange<Int> {
    override val start: Int get() = first
    override val endInclusive: Int get() = last

    override fun contains(value: Int): Boolean = first <= value && value <= last

    override fun isEmpty(): Boolean = first > last

    override fun equals(other: Any?): Boolean =
        other is IntRange && (isEmpty() && other.isEmpty() ||
        first == other.first && last == other.last)

    override fun hashCode(): Int =
        if (isEmpty()) -1 else (31 * first + last)

    override fun toString(): String = "$first..$last"

    companion object {
        /** An empty range of values of type Int. */
        public val EMPTY: IntRange = IntRange(1, 0)
    }
}
```

它的基类 `IntProgression` 实现了 `Iterable` 接口，因此 `1..10` 可以用来迭代：

``` kotlin
for (index in 1..10) {
    // 遍历 1 到 10, 包括 1 和 10.
}
```

`IntRange` 还实现了接口 `ClosedRange` ，可以用来判断某元素是否属于该区间。

Kotlin 为 `Comparable` 定义了扩展函数 `rangeTo`：

``` kotlin
/**
 * Creates a range from this [Comparable] value to the specified [that] value.
 *
 * This value needs to be smaller than [that] value, otherwise the returned range will be empty.
 * @sample samples.ranges.Ranges.rangeFromComparable
 */
public operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T> = ComparableRange(this, that)
```

因此所有的 `Comparable` 对象都可以使用 `..` 区间操作符，例如：

``` kotlin
fun main(args: Array<String>) {
    val c1 = Calendar.getInstance() // 代表今天.
    val c2 = Calendar.getInstance() 
    c2.add(Calendar.DATE, 10)       // 代表 10 天后.
    val c3 = Calendar.getInstance()
    c3.add(Calendar.DATE, 3)        // 代表 3 天后.
    val c4 = Calendar.getInstance()
    c4.add(Calendar.DATE, 13)       // 代表 13 天后.
    
    // 判断某日期是否在某两个日期范围内.
    println(c3 in c1..c2)
    println(c4 in c1..c2)
}

/*
true
false
*/
```

## 操作符函数与 Java

* Java 中调用 Kotlin 中的操作符方法，就跟调用普通方法一样，你不能期望在 Java 中写 `new Point(1, 2) + new Point(3, 4)` 这样的语法，只能乖乖调用 `new Point(1, 2).plus(new Point(3, 4))`。

* 反之，Kotlin 中调用 Java 代码却可以同 Kotlin 中自定义操作符方法一样方便。只要一个类提供了满足操作符方法签名的方法，哪怕它只是一个普通方法，不需要加 `operator` 修饰符（Java 中也没有这个修饰符），就可以在 Kotlin 中以操作符的方式调用。例如：`arrayList[0]` 相当于 Java 中 `arrayList.get(0)`，尽管这个 `get` 方法是 Java 中定义的。

* Java 中的位运算符在 Kotlin 中是没有的，它们只能使用普通方法加中缀表达式使用，只能用于 `Int` 和 `Long`，对应关系如下：

| Java 中      | Kotlin 中 |
| :----------- | :---- |
| << 有符号左移  | shl(bits) |
| >> 有符号右移  | shr(bits) |
| >>> 无符号右移 | ushr(bits) |
| & 与         | and(bits) |
| \| 或        | or(bits) |
| ^ 异或       | xor(bits) |
| ! 非         | inv() |


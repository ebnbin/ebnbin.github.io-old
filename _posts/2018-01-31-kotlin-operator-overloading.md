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

* **`operator` 修饰符是必须的**，否则 `plus` 只是一个普通方法，不能通过 `+` 调用。

* 操作符是有优先级的，比较 `*` 优先级高于 `+`，不论这个操作符应用于什么对象，这种优先级都是固定存在的。

* `plus` 方法的参数类型是任意的，因此可以方法重载，但是 **参数数量只能是 1** ，因为 `+` 是一个二元操作符。`plus` 方法的返回值类型也是任意的。

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

同时，所有操作符都针对基本类型做了优化，比如 `1 + 2 * 3`、`4 < 5`，不会为它们引入函数调用的开销。

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

* 如果执行 `a += b` 时 `plusAssign` 不存在，会尝试生成 `a = a + b`，其中的 `a + b` 使用的就是 `plus` 操作符方法，相当于调用 `a = a.plus(b)`。并且此时会 **要求 `a + b` 的 `plus` 方法的返回值类型必须与 `a` 类型一致**（如果单独使用 `a + b` 不做此要求）。

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

## in

| 表达式 | 翻译为 |
| :---- | :---- |
| a in b  | b.contains(a) |
| a !in b | !b.contains(a) |

``` kotlin
println("hello" in arrayListOf("hello", ", ", "world"))

/*
true
*/
```

**在 `for` 循环中使用 `in` 操作符会执行迭代操作，`for(x in list) { /* 遍历 */ }` 将被转换成 `list.iterator()` 的调用，然后在上面重复调用`hasNext` 和 `next` 方法。**

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

## 一元前缀操作符

| 表达式 | 翻译为 |
| :---- | :---- |
| +a | a.unaryPlus() |
| -a | a.unaryMinus() |
| !a | a.not() |

``` kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

val point = Point(10, 20)
println(-point)

/*
Point(x=-10, y=-20)
*/
```

## 递增与递减

| 表达式 | 翻译为 |
| :---- | :---- |
| a++ | a.inc() |
| a-- | a.dec() |

编译器自动支持与普通数字类型的前缀、后缀自增运算符相同的语义。例如后缀运算会先返回变量的值，然后才执行 `++` 操作。

## 索引访问操作符

| 表达式 | 翻译为 |
| :---- | :---- |
| a[i]                | a.get(i) |
| a[i, j]             | a.get(i, j) |
| a[i_1, ……, i_n]     | a.get(i_1, ……, i_n) |
| a[i] = b            | a.set(i, b) |
| a[i, j] = b         | a.set(i, j, b) |
| a[i_1, ……, i_n] = b | a.set(i_1, ……, i_n, b) |

``` kotlin
@Suppress("IMPLICIT_CAST_TO_ANY", "UNCHECKED_CAST")
operator fun <T> SharedPreferences.get(key: String, defValue: T) = when (defValue) {
    is String -> getString(key, defValue)
    is Int -> getInt(key, defValue)
    is Long -> getLong(key, defValue)
    is Float -> getFloat(key, defValue)
    is Boolean -> getBoolean(key, defValue)
    else -> throw RuntimeException()
} as T

@SuppressLint("CommitPrefEdits")
operator fun <T> SharedPreferences.set(key: String, value: T) = with(edit()) {
    when (value) {
        is String -> putString(key, value)
        is Int -> putInt(key, value)
        is Long -> putLong(key, value)
        is Float -> putFloat(key, value)
        is Boolean -> putBoolean(key, value)
        else -> throw RuntimeException()
    }.apply()
}

fun main(args: Array<String>) {
    val version = sp["key_version", 47] // 读 sp.
    sp["key_version"] = 48 // 写 sp.
}
```

## 调用操作符

| 表达式 | 翻译为 |
| :---- | :---- |
| a()             | a.invoke() |
| a(i)            | a.invoke(i) |
| a(i, j)         | a.invoke(i, j) |
| a(i_1, ……, i_n) | a.invoke(i_1, ……, i_n) |

## 相等与不等操作符

| 表达式 | 翻译为 |
| :---- | :---- |
| a == b | a?.equals(b) ?: (b === null) |
| a != b | !(a?.equals(b) ?: (b === null)) |

这在 `Any` 中被定义。**Java 的 `a.equals(b)` 相当于 Koltin 的 `a == b`，Java 的 `a == b` 相当于 Kotlin 的 `a === b`（同一性检查）**。要自定义 `==` 操作符其实就是覆写 `equals` 方法。Kotlin 中 `===` 不可被重载。

## 比较操作符

| 表达式 | 翻译为 |
| :---- | :---- |
| a > b | a.compareTo(b) > 0 |
| a < b | a.compareTo(b) < 0 |
| a >= b | a.compareTo(b) >= 0 |
| a <= b | a.compareTo(b) <= 0 |

要求 `compareTo` **返回值类型必须为 `Int`** ，这与 `Comparable` 接口保持一致。

``` kotlin
data class Movie(val name: String, val score: Int, val date: Date, val other: Any = Any()) : Comparable<Movie> {
    override fun compareTo(other: Movie): Int {
        return compareValuesBy(this, other, Movie::score, Movie::date, Movie::name) // 如果将 Movie::other 也用作比较会报错, 因为 other 不是 Comparable 类型的。
    }
}

fun main(args: Array<String>) {
    val df = SimpleDateFormat("yyyy-MM-dd", Locale.getDefault())
    val movie0 = Movie("马戏之王", 8, df.parse("2018-01-31"))
    val movie1 = Movie("神秘巨星", 7, df.parse("2018-01-01"))
    val movie2 = Movie("移动迷宫", 7, df.parse("2018-01-02"))
    
    println(movie0 < movie1)
    println(movie1 < movie2)
}

/*
false
true
*/
```

其中的 `compareValuesBy` 方法如下：

``` kotlin
/**
 * Compares two values using the specified functions [selectors] to calculate the result of the comparison.
 * The functions are called sequentially, receive the given values [a] and [b] and return [Comparable]
 * objects. As soon as the [Comparable] instances returned by a function for [a] and [b] values do not
 * compare as equal, the result of that comparison is returned.
 *
 * @sample samples.comparisons.Comparisons.compareValuesByWithSelectors
 */
public fun <T> compareValuesBy(a: T, b: T, vararg selectors: (T) -> Comparable<*>?): Int {
    require(selectors.size > 0)
    return compareValuesByImpl(a, b, selectors)
}

private fun <T> compareValuesByImpl(a: T, b: T, selectors: Array<out (T)->Comparable<*>?>): Int {
    for (fn in selectors) {
        val v1 = fn(a)
        val v2 = fn(b)
        val diff = compareValues(v1, v2)
        if (diff != 0) return diff
    }
    return 0
}
```

我们定义一个 `Movie` 类，它实现了 `Comparable` 接口，在比较时，希望按照 `评分` 、 `上映日期` 、 `电影名称` 的优先级顺序排序。可以简单的使用比较操作符对 `Movie` 对象进行“大小比较”。

## 操作符函数与 Java

* Java 中调用 Kotlin 中的操作符方法，就跟调用普通方法一样，你不能期望在 Java 中写 `new Point(1, 2) + new Point(3, 4)` 这样的语法，只能乖乖调用 `new Point(1, 2).plus(new Point(3, 4))`。

* 反之，Kotlin 中调用 Java 代码却可以同 Kotlin 中自定义操作符方法一样方便。只要一个类提供了满足操作符方法签名的方法，哪怕它只是一个普通方法，不需要加 `operator` 修饰符（Java 中也没有这个修饰符），就可以在 Kotlin 中以操作符的方式调用。例如：`arrayList[0]` 相当于 Java 中 `arrayList.get(0)`，尽管这个 `get` 方法是 Java 中定义的。又比如所有实现了 `Comparable` 的类实例都可以使用比较操作符 `>`、`<` 等进行比较。

* **Java 中的位运算符在 Kotlin 中是没有的** ，它们只能使用普通方法加中缀表达式使用，只能用于 `Int` 和 `Long`，对应关系如下：

| Java 中      | Kotlin 中 |
| :----------- | :---- |
| << 有符号左移  | shl(bits) |
| >> 有符号右移  | shr(bits) |
| >>> 无符号右移 | ushr(bits) |
| & 与         | and(bits) |
| \| 或        | or(bits) |
| ^ 异或       | xor(bits) |
| ! 非         | inv() |

## 操作符重载与属性委托、中缀调用

我们在使用委托属性时也用过 `operator` 修饰符：

``` kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        //...
    }
 
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        //...
    }
}
```

符合这样方法签名的 `getValue` 、 `setValue` 也是操作符函数，用于委托属性的 getter 和 setter。

可以看出，操作符重载并不是一定要用如 `*` 、 `+` 、 `<` 这样的符号来表示的，比如之前的 `in` 操作符，这里的 `getter` 、 `setter`。

除了以上这些标准的可被重载的操作符外，我们也可以通过中缀函数的调用来模拟自定义中缀操作符，实现形如 `a in list` 这样的语法。

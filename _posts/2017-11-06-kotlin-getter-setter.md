---
layout: post
date: 2017-11-06 10:25:39 +0800
title: Kotlin 的 Getter 和 Setter
tags:
    - Kotlin
---

关于 Kotlin 的 getter 和 setter 的基础语法就不详细介绍了。但是就是这么一个简单的概念，却有很多容易让人忽略细节。如果不理解 Kotlin 的语法在编译到字节码后是什么样的，你可能无法真正用好 getter 和 setter，先来个例子：

``` kotlin
private val foo = calcValue("foo")
private val bar get() = calcValue("bar")

private fun calcValue(name: String): Int {
    println("Calculating $name")
    return 3
}
```

是的唯一的不同是添加了 `get()`，这两个属性在访问时的行为却不一致了。

属性 `foo` 只有第一次使用时会调用 `calcValue()`，属性 `bar` 每一次使用是都会调用 `calcValue()`。可以通过反编译后的 Java 代码明确看出区别：

``` java
private final int foo = calcValue("foo");

private final int calcValue(String paramString)
{
String str = "Calculating " + paramString;
System.out.println(str);
return 3;
}

private final int getBar()
{
return calcValue("bar");
}
```

---

一个简单的访问权限修饰符也能导致其对应的 Java 代码的较大的差别：

``` kotlin
private val foo = 3
val bar = 3
```

反编译后：

``` java
private final int bar = 3;
private final int foo = 3;

public final int getBar()
{
return bar;
}
```

---

在 Kotlin 中，类中没有字段这一概念，所有形如 Java 中 `field` 的变量在 Kotlin 中都叫 **属性**，它和 **字段** 的区别是所有的属性都有对应的 getter 和 setter，`val` 属性没有 setter，`private` 属性在编译成可能省略 getter 和 setter。这体现在，Kotlin 代码中的所谓 “字段” 在 Java 文件中使用时是无法直接调用的，必须通过 getXxx() 和 setXxx(value) 调用 Kotlin 中名为 xxx 的属性。还体现在，Kotlin 代码中如果已经存在名为 xxx 的属性，就不允许再创建名为 getXxx() 或 setXxx(value) 的方法。

因此，在 Kotlin 中的一个 **有初始化值** 的属性对应了 Java 中的一个 `private` 字段加上其相应的 getter 和 setter。这个字段一定是 `private` 的，属性的修饰符体现在对应的 getter 和 setter 上。

`var` 和 `val` 的唯一区别在于 `var` 既有 getter 又有 setter，`val` 只有 getter 没有 setter。

`val` 与 Java 中的 `final` 不完全相同。只有一种情况它们完全相同，就是像 `private val foo = 3` 这样，**以 `private` 修饰**，**赋予初始值**，**并且不提供自定义 getter**，**不被委托**的属性，这个属性完全等同于 Java 中的 `private final int foo = 3;`。

---

再回顾上面第一个例子，我们把代码简化一下，关注这个 `get()`：

``` kotlin
val foo = 3

val bar
    get() = 3
```

``` java
private final int foo = 3;

public final int getBar()
{
return 3;
}

public final int getFoo()
{
return foo;
}
```

注意到，被 `private` 修饰的 Kotlin 属性在编译成字节码后省略了 getter 和 setter，因为 `private` 属性的非自定义 getter 和 setter 的存在是没有意义的，省略 getter 和 setter 是为了节省调用方法的开销。就如在 Java 中，如果你的 getter 和 setter 只是简单的赋值和返回，那么调用 getXxx() 和 setXxx(value) 和直接使用字段有什么差别呢？我个人认为 Java 中无意义的 getter 和 setter 是非常糟糕的实践，我们每个人在初学 Java 时都被告知，类里的字段不应该是 `public` 的，最好要被 `private` 修饰，然后提供对应的 getXxx() 和 setXxx(value)，所以每一个 Java 初学者都习惯了这种写法，却不知道为什么！！？？为了封装？封装什么？不还是通过 getXxx() 原样暴露出去了吗？

渐渐地我们发现，Java 中的 getter 和 setter **不是没有意义的**，**只是经常被滥用了**。只有当一个字段的 getter 和 setter **在意义上** 需要被自定义，它才有存在的必要，对于一个纯 Model，赋值取值时不做校验且需要对外暴露的字段，或者在意义上就是共享的字段，强行封装一层 getter 和 setter 只会导致冗余没有任何意义。

---

Java 中与字段分离的 getter 和 setter 有两个缺点，第一书写相对麻烦，这个不多说了，第二：

``` java
private int x;
private int y;

public int getX() {
    return x;
}

public int getY() {
    return y;
}

public void setX(int x) {
    if (this.x == x) {
        return;
    }

    this.x = x;
    onPositionChanged();
}

public void setY(int y) {
    if (this.y == y) {
        return;
    }

    this.y = y;
    onPositionChanged();
}
    
public void setPosition(int x, int y) {
    if (this.x == x && this.y == y) {
        return;
    }
    
    this.x = x;
    this.y = y;
    onPositionChanged();
}

protected void onPositionChanged() {
}
```

OK 我很有可能在类内部忘记调用 `setX(x)` 而直接使用 `x = 100`，那么 `onPositionChanged` 回调不就被我遗漏了吗？这时候有两种说辞：

* \- 我可能就是不需要 `onPositionChanged` 回调啊！\- 那在业务复杂的时候你自己不会混乱什么时候调用 x 什么时候调用 getX() 吗？它们俩为什么意义不同呢？如果真的有这个必要，不应该以更明确的方式写出来吗？
* \- 那我每次都调用 setX(x) 呗！\- 既然如此，为啥非得以这样的麻烦语法去写它？IDE 提供的 `Getter and Setter Generator` 就已经说明了——“这是个语言设计缺陷，这么麻烦的操作让我用外部工具帮你批量完成它吧！”

---

Kotlin 的 getter 和 setter 是与属性绑定的：

``` kotlin
var foo
    get() = ...
    protected set(value) = ...

val bar
    get() { return ... }
```

* 如果没有权限修饰符修饰 getter 和 setter，那么默认为属性的修饰符，getter 和 setter 的修饰符不能 **大于** 属性的修饰符，比如被 `protected` 修饰的属性的 getter 和 setter 不能被 `public` 修饰。

* 不能同时存在两个形如 `foo` 和 `Foo` 的字段，因为他们在 Java 中的 getter 和 setter 调用方法签名冲突。

* 如果 Kotlin 中的属性使用了 Java 的关键字（但非 Kotlin 的关键字，比如 default、new），那么 Kotlin 会为对应的 Java 字段添加一个前缀，比如 `jdField_`，在 Kotlin 中的 `default` 字段反编译后得到 Java 中的 `jdField_default`，但是其 getter 还是 `getDefault()`，如果在 Kotlin 中再添加一个名为 `jdField_default` 的字段，编译运行都不会出问题，但是反编译出来的 Java 文件中出现了两个重名的字段，不确定是不是我使用的反编译工具的问题。。。不过这个问题要想做兼容并不困难，具体就不再深究。代码如下：

``` kotlin
var default: Int = 0
var jdField_default: Int = 0
var new: Int = 0
```

``` java
private int jdField_default;
private int jdField_default;
private int jdField_new;

public final int getDefault()
{
return jdField_default;
}

public final int getJdField_default()
{
return jdField_default;
}

public final int getNew()
{
return jdField_new;
}

public final void setDefault(int paramInt)
{
jdField_default = paramInt;
}

public final void setJdField_default(int paramInt)
{
jdField_default = paramInt;
}

public final void setNew(int paramInt)
{
jdField_new = paramInt;
}
```

---

如果你确定你已经理解了 Kotlin 属性，来看最后这个例子：

``` kotlin
var a = 3
// var a: Int = 3
// 省略可推断类型.

var b = 3
    // Warning: Redundant setter.
    get() = field
    // Warning: Redundant setter.
    set(value) {
        field = value
    }

// 编译错误: Property must be initialized or be abstract.
// var c: Int

// 编译错误: Property must be initialized.
// var d
//     get() = 3

val e
    get() = 3

var f
    get() = 3
    // Warning: Redundant setter.
    set(value) {}
```

* 属性 `a` 和属性 `b` 在字节码层面 **完全一致**，属性 `b` 中的 getter 和 setter 写与不写完全没差。
* 对比属性 `d` 和属性 `e`，为什么 `d` 编译错误呢，因为我们上面说过属性 `b` 的 setter 完全可以省略，意味着属性 `a` 的 setter 其实就长属性 `b` 的 setter 那样，只是被省略了，那对于这样一个默认的 setter，如果属性 `e` 不初始化，setter 中的 `field` 字段从何而来？`field` 是一个代表了当前属性的 **幕后字段** ，它的作用域只限于当前 setter（getter 中也有一个 `field`），由于 Kotlin 中的类属性不像 Java 中的类字段，Kotlin 中的类属性是不提供默认初始化的，必须手动指定其默认值，因此属性 `e` 必须被初始化，setter 中的 `field` 被访问时才有值。而对于 `val` 属性 `e`，没有 setter，虽然 getter 中也可以存在 `field` 幕后字段，但是这里直接返回 `3` 没有用到 `field`，所以属性 `e` 这么写是没有问题的。
* 对比属性 `d` 和属性 `f`，多了个 setter，覆盖了默认的 `field = value` 的 setter，那么 `field` 也就没有引用了，`f` 的初始化也就可以省略了。
* 属性 `f` 的写法与直接写两个名为 `getF()` 和 `setF(value)` 的方法在字节码完全没区别。
* 像属性 `a`、属性 `b` 这种带初始化值的属性，在 Java 字节码会创建对应的字段，而像属性 `e`、属性 `f` 这样不带初始化值的属性，就不会有对应的字段被生成，原因很简单，它们不需要幕后字段 `field`，这个幕后字段其实就是对应到 Java 中的类字段，在 Kotlin 中，这样的幕后字段是用来存储属性真正的值的，除了 getter 和 setter 外部访问不到。如果 getter 和 setter 都不需要，那它就会被省略。
* 最后需要注意的一点：属性 `b` 的 setter 所带的 warning，和属性 `f` 的 setter 所带的 warning，虽然都提示 `Redundant setter.`，但它们意义不同。前者是说，“你不写这个 setter 我也会默认帮你做一样的事情 `field = value`”，后者是说，“你写了一个啥事情都没做一条语句都没有的空方法”，后者的 setter 也会被 IDE 提示“可删除”，**但是如果删除了，逻辑是不同的**。

以上这些分析，可以很容易从下面反编译出的 Java 代码得到验证：

``` java
private int a = 3;
private int b = 3;

public final int getA()
{
return a;
}

public final int getB()
{
return b;
}

public final int getE()
{
return 3;
}

public final int getF()
{
return 3;
}

public final void setA(int paramInt)
{
a = paramInt;
}

public final void setB(int paramInt)
{
b = paramInt;
}

public final void setF(int paramInt) {}
```

## 总结

* Kotlin 中的类 **属性** 对映 Java 中的类 **字段** 加上其对应的 getter 和 setter。
* 属性的访问权限修饰符作用于其 getter 和 setter，getter 和 setter 的权限不能大于属性的权限，对应的 Java 字段始终为 `private` 权限。
* `val` 属性没有 setter，形如 `private val a = 0` 的属性与 Java 中的 `private final` 字段完全相同。
* 幕后字段 `field`，用来在 getter 和 setter 中访问当前属性，相当于 Java 中的 **private 字段**。
* 一定条件下属性可以不赋予初始化值，对应在 Java 中不会生成字段，只会生成 getter 和 setter。

好了，现在随便写一个 Kotlin 属性，脑中马上就能反映出来对应的 Java 代码了吧！

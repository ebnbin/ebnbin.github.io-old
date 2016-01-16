---
layout:     post
title:      "hashCode 方法和 equals 方法"
subtitle:   ""
date:       2016-01-15 19:22:07
author:     "Ebn Zhang"
#header-img: ""
tags:
    - Java
---

`hashCode` 和 `equals` 都是 Java 中 `Object` 类的方法，在往 `HashSet` 集合类中 `add` 自定义类对象时，正确的覆写自定义类中的这两个方法是十分必要的。

## “相同”的自定义类对象

``` java
public class Test {
    public static void main(String[] args) {
        HashSet<Resolution> resolutions = new HashSet<>();
        resolutions.add(new Resolution(1920, 1080));
        resolutions.add(new Resolution(1280, 720));
        resolutions.add(new Resolution(1920, 1080));
        System.out.println(resolutions.size() + ": " + resolutions);
        // 打印：“3: [{1920, 1080}, {1920, 1080}, {1280, 720}]”
        // 别问为什么不是“3: [{1920, 1080}, {1280, 720}, {1920, 1080}]”
    }
}

class Resolution {
    public final int x;
    public final int y;

    public Resolution(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public String toString() {
        return "{" + x + ", " + y + '}';
    }
}
```

通常情况下这并不是我们想要的结果，既然用了 `HashSet` 这个集合类，我们自然希望 `x` 字段与 `y` 字段同时相等的两个 `Resolution` 对象不要被重复的添加进 `HashSet`。

## `hashCode` 方法和 `equals` 方法

其实 `HashSet` 的 `add` 方法在判断两个对象是否“相同”的依据是 `hashCode` 和 `equals` 这两个方法。为了验证，我们用下面四种方式覆写 `Resolution` 类的这两个方法：

``` java
// 方式1：都只调用父类方法（除了打印信息之外，相当于没覆写）
// 
// 打印：“
// hashCode{1920, 1080}
// hashCode{1280, 720}
// hashCode{1920, 1080}
// 3: [{1920, 1080}, {1920, 1080}, {1280, 720}]
// ”
class Resolution {
    // ...其他代码

    @Override
    public int hashCode() {
        System.out.println("hashCode" + toString());
        return super.hashCode();
    }

    @Override
    public boolean equals(Object obj) {
        System.out.println("equals" + toString());
        return super.equals(obj);
    }
}

// 方式2：只覆写 hashCode 方法，返回一个 int 常量
// 
// 打印：“
// hashCode{1920, 1080}
// hashCode{1280, 720}
// equals{1280, 720}
// hashCode{1920, 1080}
// equals{1920, 1080}
// equals{1920, 1080}
// 3: [{1920, 1080}, {1280, 720}, {1920, 1080}]
// ”
class Resolution {
    // ...其他代码

    @Override
    public int hashCode() {
        System.out.println("hashCode" + toString());
        return 100;
    }

    @Override
    public boolean equals(Object obj) {
        System.out.println("equals" + toString());
        return super.equals(obj);
    }
}

// 方式3：只覆写 equals 方法，返回一个 boolean 常量
// 
// 打印：“
// hashCode{1920, 1080}
// hashCode{1280, 720}
// hashCode{1920, 1080}
// 3: [{1920, 1080}, {1920, 1080}, {1280, 720}]
// ”
class Resolution {
    // ...其他代码

    @Override
    public int hashCode() {
        System.out.println("hashCode" + toString());
        return super.hashCode();
    }

    @Override
    public boolean equals(Object obj) {
        System.out.println("equals" + toString());
        return true;
    }
}

// 方式4：同时覆写两个方法
// 
// 打印：“
// hashCode{1920, 1080}
// hashCode{1280, 720}
// equals{1280, 720}
// hashCode{1920, 1080}
// equals{1920, 1080}
// 1: [{1920, 1080}]
// ”
class Resolution {
    // ...其他代码

    @Override
    public int hashCode() {
        System.out.println("hashCode" + toString());
        return 100;
    }

    @Override
    public boolean equals(Object obj) {
        System.out.println("equals" + toString());
        return true;
    }
}
```

从上面的代码可以看出， `HashSet` 的 `add` 方法在判断是否已存在“相同”的对象时，会首先调用对象的 `hashCode` 方法判断是否已存在相同的值，如果不存在，就将这个对象当作与其他已存在的对象“不相同”的对象添加入集合，否则，会调用对象的 `equals` 方法与 `hashCode` 值相同的对象比较，如果返回 `true`，就会被当作集合中已存在“相同”对象而不会再次被添加入集合，否则会继续调用 `equals` 方法与下一个 `hashCode` 值相同的对象比较，直到与所有 `hashCode` 值相同的对象比较后返回的都是 `false`，才会添加这个对象到集合中（**请注意：在方式2中，`main` 方法打印的信息之前，`equals{1920, 1080}` 被打印了两次**）。

简单的说，`HashSet` 的 `add` 方法判断两个对象是否“相同”时：
1. 判断两个对象的hashCode是否相等；
    * 如果不相等，认为两个对象“不相同”，添加到集合中，结束。
    * 如果相等，转入2；
2. 判断两个对象用equals运算是否相等；
    * 如果不相等，认为两个对象“不相同”，添加到集合中。
    * 如相等，认为两个对象“相同”，不添加到集合中。

由 `Object` 类定义的 `hashCode` 方法会针对不同的对象返回不同的整数（可以大体理解为通过将该对象的内部地址转换成一个整数来实现的，虽然这样的实现技巧的描述并不准确），而 `Object` 类的 `equals` 方法直接返回的就是两个对象的引用比较，对于不同的对象，`equals` 方法返回的一定是 `false`，所以如果自定义类不同时覆写 `hashCode` 方法和 `equals` 方法，该类的所有对象就都会被当成“不相同”的对象而被添加进 `HashSet`。

## 解决方法

那么就来以合适的方式覆写一下这两个方法吧。

```java
public class Test {
    public static void main(String[] args) {
        HashSet<Resolution> resolutions = new HashSet<>();
        resolutions.add(new Resolution(1920, 1080));
        resolutions.add(new Resolution(1280, 720));
        resolutions.add(new Resolution(1920, 1080));
        System.out.println(resolutions.size() + ": " + resolutions);
        // 打印：“2: [{1920, 1080}, {1280, 720}]”
    }
}

class Resolution {
    public final int x;
    public final int y;

    public Resolution(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public String toString() {
        return "{" + x + ", " + y + '}';
    }

    @Override
    public int hashCode() {
        return x * 32713 + y;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Resolution)) {
            return false;
        }
        Resolution resolution = (Resolution) obj;
        return x == resolution.x && y == resolution.y;
    }
}
```

一定有人问 `hashCode` 方法中的 `32713` 这个数字是什么意思。**`hashCode` 方法是为了提高判断效率而要求的**，这句话在这里就得到了很好的体现。在大部分的情况下，只要 `x` 和 `y` 都不相同，`hashCode` 也不会重复，在极少数的情况下，比如 `new Resolution(1, 32713)` 和 `new Resolution(2, 0)` 这两个对象的 `hashCode` 就发生了重复，但是这并不要紧，因为 **`equals` 方法才是判断两个对象是否相等的关键**，在 `equals` 方法中具体的判断了两个对象的 `x` 和 `y` 必须同时相等才视两个对象“相同”。所以， `hashCode` 方法中的 `32713` 数字完全只是为了在大部分情况下快速的判断两个对象是否“相同”，它可以是 `0`、`47`、`-52`、`1415926535`，甚至 `hashCode` 可以写成 `return x + y;` 或者 `return x;`，只不过这样增大了 `hashCode` 重复的可能性，也就有更多的可能需要多花时间去调用 `equals` 方法进行判断了。通常情况下，`32713` 这样的数字作为基数，使用一个在合适范围内的质数是最好的。

---
layout:     post
title:      "Java 8 默认方法（Default Methods）"
subtitle:   ""
date:       2015-12-20 22:15:52
author:     "Ebn Zhang"
#header-img: ""
tags:
    - Java
---

Java 8 引入了新的语言特性——默认方法（Default Methods）。

> [Default methods](http://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html) enable new functionality to be added to the interfaces of libraries and ensure binary compatibility with code written for older versions of those interfaces.

> 默认方法允许您添加新的功能到现有库的接口中，并能确保与采用旧版本接口编写的代码的二进制兼容性。

默认方法是在接口中的方法签名前加上了 `default` 关键字的实现方法。

## 一个简单的例子

``` java
interface InterfaceA {
    default void foo() {
        System.out.println("InterfaceA foo");
    }
}

class ClassA implements InterfaceA {
}

public class Test {
    public static void main(String[] args) {
        new ClassA().foo(); // 打印：“InterfaceA foo”
    }
}
```

`ClassA` 类并没有实现 `InterfaceA` 接口中的 `foo` 方法，`InterfaceA` 接口中提供了 `foo` 方法的默认实现，因此可以直接调用 `ClassA` 类的 `foo` 方法。

## 为什么要有默认方法

在 java 8 之前，接口与其实现类之间的 **耦合度** 太高了（**tightly coupled**），当需要为一个接口添加方法时，所有的实现类都必须随之修改。默认方法解决了这个问题，它可以为接口添加新的方法，而不会破坏已有的接口的实现。这在 lambda 表达式作为 java 8 语言的重要特性而出现之际，为升级旧接口且保持向后兼容（backward compatibility）提供了途径。

``` java
String[] array = new String[] {
        "hello",
        ", ",
        "world",
};
List<String> list = Arrays.asList(array);
list.forEach(System.out::println); // 这是 jdk 1.8 新增的接口默认方法
```

这个 `forEach` 方法是 jdk 1.8 新增的接口默认方法，正是因为有了默认方法的引入，才不会因为 `Iterable` 接口中添加了 `forEach` 方法就需要修改所有 `Iterable` 接口的实现类。

下面的代码展示了 jdk 1.8 的 `Iterable` 接口中的 `forEach` 默认方法：

``` java
package java.lang;

import java.util.Objects;
import java.util.function.Consumer;

public interface Iterable<T> {
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
}
```

## 默认方法的继承

和其它方法一样，接口默认方法也可以被继承。

``` java
interface InterfaceA {
    default void foo() {
        System.out.println("InterfaceA foo");
    }
}

interface InterfaceB extends InterfaceA {
}

interface InterfaceC extends InterfaceA {
    @Override
    default void foo() {
        System.out.println("InterfaceC foo");
    }
}

interface InterfaceD extends InterfaceA {
    @Override
    void foo();
}

public class Test {
    public static void main(String[] args) {
        new InterfaceB() {}.foo(); // 打印：“InterfaceA foo”
        new InterfaceC() {}.foo(); // 打印：“InterfaceC foo”
        new InterfaceD() {
            @Override
            public void foo() {
                System.out.println("InterfaceD foo");
            }
        }.foo(); // 打印：“InterfaceD foo”
        
        // 或者使用 lambda 表达式
        ((InterfaceD) () -> System.out.println("InterfaceD foo")).foo();
    }
}
```

接口默认方法的继承分三种情况（分别对应上面的 `InterfaceB` 接口、`InterfaceC` 接口和 `InterfaceD` 接口）：

* 不覆写默认方法，直接从父接口中获取方法的默认实现。

* 覆写默认方法，这跟类与类之间的覆写规则相类似。

* 覆写默认方法并将它重新声明为抽象方法，这样新接口的子类必须再次覆写并实现这个抽象方法。

## 默认方法的多继承

Java 使用的是单继承、多实现的机制，为的是避免多继承带来的调用歧义的问题。当接口的子类同时拥有具有相同签名的方法时，就需要考虑一种解决冲突的方案。

``` java
interface InterfaceA {
    default void foo() {
        System.out.println("InterfaceA foo");
    }
}

interface InterfaceB {
    default void bar() {
        System.out.println("InterfaceB bar");
    }
}

interface InterfaceC {
    default void foo() {
        System.out.println("InterfaceC foo");
    }
    
    default void bar() {
        System.out.println("InterfaceC bar");
    }
}

class ClassA implements InterfaceA, InterfaceB {
}

// 错误
//class ClassB implements InterfaceB, InterfaceC {
//}

class ClassB implements InterfaceB, InterfaceC {
    @Override
    public void bar() {
        InterfaceB.super.bar(); // 调用 InterfaceB 的 bar 方法
        InterfaceC.super.bar(); // 调用 InterfaceC 的 bar 方法
        System.out.println("ClassB bar"); // 做其他的事
    }
}
```

在 `ClassA` 类中，它实现的 `InterfaceA` 接口和 `InterfaceB` 接口中的方法不存在歧义，可以直接多实现。

在 `ClassB` 类中，它实现的 `InterfaceB` 接口和 `InterfaceC` 接口中都存在相同签名的 `foo` 方法，需要手动解决冲突。覆写存在歧义的方法，并可以使用 `InterfaceName.super.methodName();` 的方式手动调用需要的接口默认方法。

## 接口继承行为发生冲突时的解决规则

值得注意的是这么一种情况：

``` java
interface InterfaceA {
    default void foo() {
        System.out.println("InterfaceA foo");
    }
}

interface InterfaceB extends InterfaceA {
    @Override
    default void foo() {
        System.out.println("InterfaceB foo");
    }
}

// 正确
class ClassA implements InterfaceA, InterfaceB {
}

class ClassB implements InterfaceA, InterfaceB {
    @Override
    public void foo() {
//        InterfaceA.super.foo(); // 错误
        InterfaceB.super.foo();
    }
}
```

当 `ClassA` 类多实现 `InterfaceA` 接口和 `InterfaceB` 接口时，不会出现方法名歧义的错误。当 `ClassB` 类覆写 `foo` 方法时，无法通过 `InterfaceA.super.foo();` 调用 `InterfaceA` 接口的 `foo` 方法。

因为 `InterfaceB` 接口继承了 `InterfaceA` 接口，那么 `InterfaceB` 接口一定包含了所有 `InterfaceA` 接口中的字段方法，因此一个同时实现了 `InterfaceA` 接口和 `InterfaceB` 接口的类与一个只实现了 `InterfaceB` 接口的类完全等价。

这很好理解，就相当于 `class SimpleDateFormat extends DateFormat` 与 `class SimpleDateFormat extends DateFormat, Object` 等价（如果允许多继承）。

或者换种方式理解：

``` java
class ClassC {
    public void foo() {
        System.out.println("ClassC foo");
    }
}

class ClassD extends ClassC {
    @Override
    public void foo() {
        System.out.println("ClassD foo");
    }
}

public class Test {
    public static void main(String[] args) {
        ClassC classC = new ClassD();
        classC.foo(); // 打印：“ClassD foo”
    }
}
```

这里的 `classC.foo();` 同样调用的是 `ClassD` 类中的 `foo` 方法，打印结果为“`ClassD foo`”，因为 `ClassC` 类中的 `foo` 方法在 `ClassD` 类中被覆写了。

在上面的 `ClassA` 类中不会出现方法名歧义的原因是所谓“存在歧义”的方法其实都来自于 `InterfaceA` 接口，`InterfaceB` 接口中的“同名方法”只是继承自 `InterfaceA` 接口而来并对其进行了覆写。`ClassA` 类实现的两个接口不是两个毫不相干的接口，因此不存在同名歧义方法。

而覆写意味着对父类方法的屏蔽，这也是 `Override` 的设计意图之一。因此在实现了 `InterfaceB` 接口的类中无法访问已被覆写的 `InterfaceA` 接口中的 `foo` 方法。

这是当接口继承行为发生冲突时的规则之一，即 **被其它类型所覆盖的方法会被忽略**。

如果想要调用 `InterfaceA` 接口中的 `foo` 方法，只能通过自定义一个新的接口同样继承 `InterfaceA` 接口并显示地覆写 `foo` 方法，在方法中使用 `InterfaceA.super.foo();` 调用 `InterfaceA` 接口的 `foo` 方法，最后让实现类同时实现 `InterfaceB` 接口和自定义的新接口，代码如下：

``` java
interface InterfaceA {
    default void foo() {
        System.out.println("InterfaceA foo");
    }
}

interface InterfaceB extends InterfaceA {
    @Override
    default void foo() {
        System.out.println("InterfaceB foo");
    }
}

interface InterfaceC extends InterfaceA {
    @Override
    default void foo() {
        InterfaceA.super.foo();
    }
}

class ClassA implements InterfaceB, InterfaceC {
    @Override
    public void foo() {
        InterfaceB.super.foo();
        InterfaceC.super.foo();
    }
}
```

**注意！** 虽然 `InterfaceC` 接口的 `foo` 方法只是调用了一下父接口的默认实现方法，但是这个覆写 **不能省略**，否则 `InterfaceC` 接口中继承自 `InterfaceA` 接口的隐式的 `foo` 方法同样会被认为是被 `InterfaceB` 接口覆写了而被屏蔽，会导致调用 `InterfaceC.super.foo()` 时出错。

通过这个例子，应该注意到在使用一个默认方法前，一定要考虑它是否真的需要。因为 **默认方法会带给程序歧义，并且在复杂的继承体系中容易产生编译错误**。滥用默认方法可能给代码带来意想不到、莫名其妙的错误。

## 接口与抽象类

当接口继承行为发生冲突时的另一个规则是，**类的方法声明优先于接口默认方法，无论该方法是具体的还是抽象的**。

``` java
interface InterfaceA {
    default void foo() {
        System.out.println("InterfaceA foo");
    }

    default void bar() {
        System.out.println("InterfaceA bar");
    }
}

abstract class AbstractClassA {
    public abstract void foo();

    public void bar() {
        System.out.println("AbstractClassA bar");
    }
}

class ClassA extends AbstractClassA implements InterfaceA {
    @Override
    public void foo() {
        InterfaceA.super.foo();
    }
}

public class Test {
    public static void main(String[] args) {
        ClassA classA = new ClassA();
        classA.foo(); // 打印：“InterfaceA foo”
        classA.bar(); // 打印：“AbstractClassA bar”
    }
}
```

`ClassA` 类中并不需要手动覆写 `bar` 方法，因为优先考虑到 `ClassA` 类继承了的 `AbstractClassA` 抽象类中存在对 `bar` 方法的实现，同样的因为 `AbstractClassA` 抽象类中的 `foo` 方法是抽象的，所以在 `ClassA` 类中必须实现 `foo` 方法。

虽然 Java 8 的接口的默认方法就像抽象类，能提供方法的实现，但是他们俩仍然是 **不可相互代替的**：

* 接口可以被类多实现（被其他接口多继承），抽象类只能被单继承。

* 接口中没有 `this` 指针，没有构造函数，不能拥有实例字段（实例变量）或实例方法，无法保存 **状态**（**state**），抽象方法中可以。

* 抽象类不能在 java 8 的 lambda 表达式中使用。

* 从设计理念上，接口反映的是 **“like-a”** 关系，抽象类反映的是 **“is-a”** 关系。

## 接口静态方法

除了默认方法，Java 8 还在允许在接口中定义静态方法。

``` java
interface InterfaceA {
    default void foo() {
        printHelloWorld();
    }
    
    static void printHelloWorld() {
        System.out.println("hello, world");
    }
}

public class Test {
    public static void main(String[] args) {
        InterfaceA.printHelloWorld(); // 打印：“hello, world”
    }
}
```

## 其他注意点

* `default` 关键字只能在接口中使用（以及用在 `switch` 语句的 `default` 分支），不能用在抽象类中。

* 接口默认方法不能覆写 `Object` 类的 `equals`、`hashCode` 和 `toString` 方法。

* 接口中的静态方法必须是 `public` 的，`public` 修饰符可以省略，`static` 修饰符不能省略。

* 即使使用了 java 8 的环境，一些 IDE 仍然可能在一些代码的实时编译提示时出现异常的提示（例如无法发现 java 8 的语法错误），因此不要过度依赖 IDE。

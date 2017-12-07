---
layout: post
date: 2017-12-07 22:38:55 +0800
title: Kotlin 日常踩坑
tags:
    - Kotlin
    - Android
---

踩到一个 Koltin 加 Android api 27 的坑。

Support library 升级到 27.x.x 之后，发现项目一堆报错：

![error](/img/in-post/kotlin-trap/error.png)

它说我的 `onCreateView` 方法覆写了 nothing。**Are You Kidding me?**

对比了一下两个版本的 support library，发现了问题。

27.x.x 版本：
```java
@Nullable
public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container,
        @Nullable Bundle savedInstanceState) {
    return null;
}
```

26.x.x 版本
```java
@Nullable
public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container,
        @Nullable Bundle savedInstanceState) {
    return null;
}
```

就是这个 `LayoutInflater` 前的 `@NonNull` 注解导致的，在 kotlin 中，任何类型都分 nullable 和 non-null 两种，并且不能隐式转换，在 kotlin 调用 java 代码时如果 java 代码没有使用注解声明一个变量是否可为空，例如 `String str`，kotlin 会将这个变量标记为 `String!` 类型，表示我也不知道这个变量是否可为空，使用时需要自行处理，当 java 代码中使用了 `@NonNull` 和 `@Nullable` 注解标注了一个变量的类型，那么 kotlin 就明确地知道了它要么是 `String` 不可为空，要么是 `String?` 可为空。`String` 和 `String?` 可以理解为两个不相同的类型，因此上面的方法覆写就会报错，认为方法签名不相同，不能使用 `override` 关键字。

同理，在 fragment 中：

```java
@Nullable
public Context getContext() {
    return mHost == null ? null : mHost.getContext();
}
```

因此在调用 context 的方法时必须 `context!!.getString()` 而不能 `context.getString()`。

***WTTTTTTTF!***

即便在我明确认定 context 不为空的情况下，仍然需要加 `!!` 使用。于是我想了一个办法：

```kotlin
override fun getContext() = super.getContext()!!

fun getNullableContext() = super.getContext()
```

这样我就可以在明确知道 context 不为空的地方直接调用 `context.getString()` 啦，需要判断是否为空的地方调用 `getNullableContext()`。

然而问题并没有结束，因为有些地方我需要使用 `getActivity()` 而不只是 `getContext()`，可是：

```java
@Nullable
final public FragmentActivity getActivity() {
    return mHost == null ? null : (FragmentActivity) mHost.getActivity();
}
```

注意到那个“万恶”的 `final` 了吗？

***WTTTTTTTTTTTTTTF!***

只能在每个使用 `getActivity()` 并确定它不为空的地方手动加 `!!` 了。。。暂时没有想到更好的解决办法。

最后说两句：

* 第一，新的变动并不是为了给写代码添加麻烦而加上的，要明确 fragment 中的 `getContext()`、`getActivity()` 在一些情况下**就是有可能为空的**，不要简单地在所有地方加上 `!!` 就算完事，该判断的地方还是得判断的。

* 第二，虽然 27 版本的升级带来了这样的兼容问题，但整体而言，对于是否可为空类型在 java 和 kotlin 之间的调用语法更加完善了。兼容适配的过程很痛苦，但是 —— **忍着！** - -#

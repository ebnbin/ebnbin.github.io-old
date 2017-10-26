---
layout: post
date: 2017-10-23 17:30:29 +0800
title: Kotlin 委托属性
tags:
    - Kotlin
    - Android
---

简单的说，委托属性就是将一个属性的操作委托给一个委托类的实例处理，多个属性可以委托给同一个委托类。

跟没说一样。。

## 委托类

先看一个简单的例子。

``` kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "${property.name}: $thisRef"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("value=$value")
    }
}

class Test {
    var s: String by Delegate()
}

val test = Test()
println(test.s) // s: Test@4eec7777
test.s = "hello" // value=hello
```

* 委托类类名任意。

* 如果被 `val` 属性委托，必须提供 `getValue` 方法，如果被 `var` 属性委托，必须提供 `getValue` 和 `setValue` 方法。委托类中的其他属性和方法任意。

* `getValue`，`setValue` 的方法签名，参考对应接口：`kotlin.properties.ReadOnlyProperty` 和 `kotlin.properties.ReadWriteProperty`。

## `thisRef` 参数

官方对 `thisRef` 参数的要求：

> thisRef —— 必须与 **属性所有者** 类型（对于扩展属性——指被扩展的类型）相同或者是它的超类型

``` kotlin
class DelegateTest {
    private var s1: String by Delegate()
    fun test1() {
        println(s1)
    }

    fun test2() {
        var s2: String by Delegate()
        println(s2)
    }

    companion object {
        private var s3: String by Delegate()
        fun test3() {
            println(s3)
        }

        fun test4() {
            var s4: String by Delegate()
            println(s4)
        }
    }
}

private var s5: String by Delegate()
fun test5() {
    println(s5)
}

fun test6() {
    var s6: String by Delegate()
    println(s6)
}
```

测试：

``` kotlin
val test = DelegateTest()
test.test1()
test.test2()
DelegateTest.test3()
DelegateTest.test4()
test5()
test6()
```

Java 调用方式：

``` java
DelegateTest test = new DelegateTest();
test.test1();
test.test2();
DelegateTest.Companion.test3();
DelegateTest.Companion.test4();
DelegateTestKt.test5();
DelegateTestKt.test6();
```

输出：

```
s1: DelegateTest@7229724f
s2: null
s3: DelegateTest$Companion@4c873330
s4: null
s5: null
s6: null
```

可以看出，对于局部属性（在方法中声明的属性）或静态属性，`thisRef` 为 `null`，否则为属性所在对象。

## 委托类中的 `setValue`

如果委托一个 `var` 属性，希望保存属性上一次 `setValue` 的值，需要手动添加一个变量用于记录。

``` kotlin
class WeirdDelegate(initValue: String = "") {
    private var localValue: String = initValue

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "This is NOT $localValue"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        localValue = value
    }
}

var fruit by WeirdDelegate("banana")
fruit = "apple"
println(fruit) // This is NOT apple
```

## Kotlin 标准委托

Kotlin 自带一些默认的委托实现。

#### 延迟属性 Lazy

``` kotlin
val lazyValue: String by lazy {
    print("Calculating...")
    "world"
}

println(lazyValue) // Calculating...world
println(lazyValue) // world
```

#### 可观察属性 Observable

``` kotlin
class User {
    var name: String by Delegates.observable("<no name>") {
        _, old, new ->
        println("$old -> $new")
    }
}

val user = User()
user.name = "Steve" // <no name> -> Steve
user.name = "Tim" // Steve -> Tim
```

## 小结

我们可以这么理解：因为 Kotlin 中类不能有字段，只有属性，`val` 声明的是只一个有 getter 没有 setter 的属性，`var` 声明的是一个既有 getter 又有 setter 的属性，可以通过：

``` kotlin
var foo: Data
    get() { ... }
    set(value) { ... }
```

的方式自定义一个类属性的 getter 和 setter。如果有一批属性，他们都需要相同而复杂的 getter 和 setter，就可以通过委托属性实现，一个委托类可以帮助被委托的属性处理复杂的自定义 getter 和 setter 操作。

## 委托属性在 Android SharedPreferences 中的应用

#### Java 版本

通常，我们这么写一个 SharedPreferences 工具类：

``` java
public final class PreferencesUtil {
    private static PreferencesUtil sInstance;

    public static void init(Context context) {
        if (sInstance == null) {
            sInstance = new PreferencesUtil(context);
        }
    }

    public static PreferencesUtil getInstance() {
        if (sInstance == null) throw new RuntimeException("Uninitialized.");

        return sInstance;
    }

    private final SharedPreferences mSp;

    private PreferencesUtil(Context context) {
        mSp = PreferenceManager.getDefaultSharedPreferences(context);
    }

    public String getString(String key, String defValue) {
        return mSp.getString(key, defValue);
    }

    public void putString(String key, String value) {
        mSp.edit().putString(key, value).apply();
    }

    public int getInt(String key, int defValue) {
        return mSp.getInt(key, defValue);
    }

    public void putInt(String key, int value) {
        mSp.edit().putInt(key, value).apply();
    }

    public long getLong(String key, long defValue) {
        return mSp.getLong(key, defValue);
    }

    public void putLong(String key, long value) {
        mSp.edit().putLong(key, value).apply();
    }

    public float getFloat(String key, float defValue) {
        return mSp.getFloat(key, defValue);
    }

    public void putFloat(String key, float value) {
        mSp.edit().putFloat(key, value).apply();
    }

    public boolean getBoolean(String key, boolean defValue) {
        return mSp.getBoolean(key, defValue);
    }

    public void putBoolean(String key, boolean value) {
        mSp.edit().putBoolean(key, value).apply();
    }
}
```

然后这么用：

``` java
if (PreferencesUtil.getInstance().getBoolean(Constant.KEY_IS_FIRST_LAUNCH, Constant.DEF_IS_FIRST_LAUNCH)) {
    // Do something first launch, like showing Welcome.
    ...

    PreferencesUtil.getInstance().putBoolean(Constant.KEY_IS_FIRST_LAUNCH, true);
}
```

* 创建一个 `PreferencesUtil` 单例，在 `Application` 中调用 `PreferencesUtil.init(context)` 初始化。

* 代理 `SharedPreferences` 中的所有 `getXxx()`，`putXxx()` 方法，方便使用时不需要写 `getSharedPreferences().edit().putXxx().apply()` 这么长的代码。

* 使用时调用 `PreferencesUtil.getInstance().getXxx(key, defValue)` 读取，调用 `PreferencesUtil.getInstance().putXxx(key, value)` 写入。

* 写起来仍然很麻烦。每次 `getInstance()`，传 `key`，如果写入不同的 `SharedPreferences` 文件，还需要每次传文件名。

* `getXxx()` 和 `putXxx()` 要保证 `key` 相同，会去字符串常量类中找，可能出错。

* 处理的是 `SharedPreferences` 文件中的同一个 `key` 对应的 `value`，用的却是两次没有关联的 util 操作。

#### Kotlin 委托属性版本

``` kotlin
class PreferencesDelegate<T>(private val key: String, private val defValue: T) {
    private val sp by lazy { PreferenceManager.getDefaultSharedPreferences(AppApplication.instance) }

    @Suppress("IMPLICIT_CAST_TO_ANY", "UNCHECKED_CAST")
    operator fun getValue(thisRef: Any?, property: KProperty<*>) = with(sp) {
        when (defValue) {
            is String -> getString(key, defValue)
//            is Set<*> -> getStringSet(key, defValue as Set<String>) // Unsupported.
            is Int -> getInt(key, defValue)
            is Long -> getLong(key, defValue)
            is Float -> getFloat(key, defValue)
            is Boolean -> getBoolean(key, defValue)
            else -> throw RuntimeException("Unsupported type.")
        } as T
    }

    @SuppressLint("CommitPrefEdits")
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: T) = with(sp.edit()) {
        when (value) {
            is String -> putString(key, value)
            is Int -> putInt(key, value)
            is Long -> putLong(key, value)
            is Float -> putFloat(key, value)
            is Boolean -> putBoolean(key, value)
            else -> throw RuntimeException("Unsupported type.")
        }.apply()
    }
}
```

使用：

``` kotlin
private var isFirstLaunch by PreferencesDelegate(Constant.KEY_IS_FIRST_LAUNCH, Constant.DEF_IS_FIRST_LAUNCH)

if (isFirstLaunch) {
    // Do something first launch, like showing Welcome.
    ...

    isFirstLaunch = false
}
```

* 创建一个 `PreferencesDelegate` 代理类，处理各个类型的 Preferences 的存取。

* 在使用时，创建一个代表要处理的 Preference 的对应类型的属性，使用 `by Delegate` 语法，用 `PreferencesDelegate` 类代理这个属性。

* 直接对变量取值就是从 SharedPreferences 文件中读取，对变量赋值即写入 SharedPreferences。

* `IMPLICIT_CAST_TO_ANY`：这是 Kotlin 中使用 `when` 表达式时，当多个分支返回不同的类型时出现的 warning，表示 `when` 表达式的返回值类型被隐式转化成了 `Any`。

* `UNCHECKED_CAST`：范型强转 warning。

* `CommitPrefEdits`：使用 `when` 表达式时，静态分析无法判断 `SharedPreferences.Editor` 是否执行了 `commit` 调用。

* 如果委托给一个局部变量，可能出现 `UNUSED_VALUE` warning，即变量赋值后未被使用，但实际上委托类执行了写 SharedPreferences 操作，并不是无用赋值。

#### 多个 SharedPreferences 文件

如果需要存取多个 SharedPreferences 文件，可以创建多个对应的委托类，继承子一个默认的 `DefaultPreferencesDelegate`。

``` kotlin
open class DefaultPreferencesDelegate<T>(private val key: String, private val defValue: T) {
    /**
     * SharedPreferences file name. `<packageName>_preferences.xml`.
     */
    protected open val name: String = "${AppApplication.instance.packageName}_preferences"
    private val sp by lazy { AppApplication.instance.getSharedPreferences(name, Context.MODE_PRIVATE) }

    ...
}

class SettingsPreferencesDelegate<T>(key: String, defValue: T) : DefaultPreferencesDelegate<T>(key, defValue) {
    override val name = "settings"
}

class DataPreferencesDelegate<T>(key: String, defValue: T) : DefaultPreferencesDelegate<T>(key, defValue) {
    override val name = "data"
}
```

#### 从 SharedPreferences 中移除 `key`

通过 `sp.edit().remove(key).apply` 移除一个 `key`。一个简单的实现方式：

``` kotlin
operator fun setValue(thisRef: Any?, property: KProperty<*>, value: T?) = with(sp.edit()) {
    when (value) {
        null -> remove(key)
        is String -> putString(key, value)
        ...
    }.apply()
}

var content: String? by DataPreferencesDelegate(Constant.KEY_TODO, Constant.DEF_TODO)
content = null
```

这么写的不方便之处在于，这个属性可能在意义上是不可为空类型的，或者是 `Int`、`Boolean` 等类型，那么将它指定为可为空类型就不合适。如果需要更好的扩展，另写工具类支持。

#### 读取 SharedPreferences 时指定默认值为 `null`

因为使用范型实现，并且通过 `defValue` 判断要存取的 SharedPreferences 的数据类型，因此这种委托写法不支持 `sp.getXxx(key, defValue)` 时 `defValue` 为 `null`。

## Sample

最后上一段完整的 sample 代码：

``` kotlin
// PreferencesDelegates.kt

open class DefaultPreferencesDelegate<T>(private val key: String, private val defValue: T) {
    /**
     * SharedPreferences file name.
     */
    protected open val name: String = "${AppApplication.instance.packageName}_preferences"

    private val sp by lazy { AppApplication.instance.getSharedPreferences(name, Context.MODE_PRIVATE) }

    @Suppress("IMPLICIT_CAST_TO_ANY", "UNCHECKED_CAST")
    operator fun getValue(thisRef: Any?, property: KProperty<*>) = with(sp) {
        when (defValue) {
            is String -> getString(key, defValue)
            is Int -> getInt(key, defValue)
            is Long -> getLong(key, defValue)
            is Float -> getFloat(key, defValue)
            is Boolean -> getBoolean(key, defValue)
            else -> throw RuntimeException("Unsupported type.")
        } as T
    }

    @SuppressLint("CommitPrefEdits")
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: T?) = with(sp.edit()) {
        when (value) {
            null -> remove(key)
            is String -> putString(key, value)
            is Int -> putInt(key, value)
            is Long -> putLong(key, value)
            is Float -> putFloat(key, value)
            is Boolean -> putBoolean(key, value)
            else -> throw RuntimeException("Unsupported type.")
        }.apply()
    }
}

class SettingsPreferencesDelegate<T>(key: String, defValue: T) : DefaultPreferencesDelegate<T>(key, defValue) {
    override val name = "settings"
}

class DataPreferencesDelegate<T>(key: String, defValue: T) : DefaultPreferencesDelegate<T>(key, defValue) {
    override val name = "data"
}
```

``` kotlin
object PreferencesHelper {
    /**
     * Keys.
     */
    private const val KEY_IS_FIRST_LAUNCH = "is_first_launch"
    private const val KEY_TODO = "todo"

    /**
     * Default values.
     */
    private const val DEF_IS_FIRST_LAUNCH = true
    private const val DEF_TODO = "Learn Kotlin."

    var isFirstLaunch: Boolean by SettingsPreferencesDelegate(KEY_IS_FIRST_LAUNCH, DEF_IS_FIRST_LAUNCH)
    var todo: String? by DataPreferencesDelegate(KEY_TODO, DEF_TODO)
}
```

``` kotlin
import kotlinx.android.synthetic.main.activity_main.todoEditText

class MainActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)

        if (PreferencesHelper.isFirstLaunch) {
            // Do something first launch, like showing Welcome.
            showWelcome()
            PreferencesHelper.isFirstLaunch = false
        }

        // Read from SharedPreferences.
        todoEditText.setText(PreferencesHelper.todo)
    }

    private fun showWelcome() {
        AlertDialog.Builder(this)
                .setMessage("Welcome!")
                .setPositiveButton("Fine", null)
                .show()
    }

    fun saveOnClick(view: View) {
        // Write to SharedPreferences.
        PreferencesHelper.todo = todoEditText.text.toString()

        Toast.makeText(this, "Save success", Toast.LENGTH_SHORT).show()
    }

    fun clearOnClick(view: View) {
        // Remove from SharedPreferences.
        PreferencesHelper.todo = null

        todoEditText.text.clear()
        Toast.makeText(this, "Clear success", Toast.LENGTH_SHORT).show()
    }
}
```

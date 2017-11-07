---
layout: post
date: 2017-11-07 17:05:57 +0800
title: Kotlin 对 Android APK 包大小的影响
tags:
    - Kotlin
    - Android
---

使用两个空项目测试以 Java 和 Kotlin 作为开发语言生成的 APK 包的大小。

两个项目都没有任何 `res` 资源文件（ `AndroidManifest` 文件中全部使用默认值），都只有一个空的 `MainActivity` 文件，没有任何逻辑代码。Kotlin 项目中添加了 `kotlin-android` 和 `kotlin-android-extensions` 插件，引用了 `org.jetbrains.kotlin:kotlin-stdlib-jre8:$kotlin_version` 库。Java 项目中不引用任何库。`build.gradle` 文件中使用 `minifyEnabled true` 控制代码压缩，使用 `useProguard false` 控制代码混淆，分别打 Release 包，测试结果如下（大小分别为 `.apk` 文件大小和 `.dex` 文件大小）：

|            | Java      | Kotlin      |
| :--------- |:---------:| :----------:|
| 不压缩不混淆 | 5K (1K)   | 360K (1.5M) |
| 压缩不混淆   | 5K (484B) | 20K (5K)    |
| 压缩混淆     | 5K (432B) | 18K (436B)  |

应该可以理解为，Kotlin 语言的引入对 APK 包大小的影响，最大为 360K 左右，即 Kotlin 依赖库中的所有文件在项目中都用到了。一般情况下，使用 Kotlin 开发只会增加 APK 包 100~200K 的大小。其他的可能的影响就是 Kotlin 的语法糖对应的 Java 代码的复杂程度了。

比如下面这个例子 *（以下测试代码都是在不压缩不混淆的配置下反编译的）*：

``` kotlin
import kotlinx.android.synthetic.main.activity_main.helloButton

class MainActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)

        helloButton.text = "Hello"
        helloButton.setOnClickListener { Toast.makeText(this, "World", Toast.LENGTH_SHORT).show() }
    }
}
```

这个扩展插件 `Kotlin Android Extensions` 简化了 `findViewById` 的使用。

反编译后得到的 Java 代码：

``` java
public final class MainActivity
  extends Activity
{
  private HashMap _$_findViewCache;
  
  public MainActivity() {}
  
  public void _$_clearFindViewByIdCache()
  {
    if (_$_findViewCache != null) {
      _$_findViewCache.clear();
    }
  }
  
  public View _$_findCachedViewById(int paramInt)
  {
    if (_$_findViewCache == null) {
      _$_findViewCache = new HashMap();
    }
    View localView = (View)_$_findViewCache.get(Integer.valueOf(paramInt));
    if (localView == null)
    {
      localView = findViewById(paramInt);
      _$_findViewCache.put(Integer.valueOf(paramInt), localView);
    }
    return localView;
  }
  
  protected void onCreate(@Nullable Bundle paramBundle)
  {
    super.onCreate(paramBundle);
    setContentView(2130837504);
    ((Button)_$_findCachedViewById(R.id.helloButton)).setText((CharSequence)"Hello");
    ((Button)_$_findCachedViewById(R.id.helloButton)).setOnClickListener((View.OnClickListener)new View.OnClickListener()
    {
      public final void onClick(View paramAnonymousView)
      {
        Toast.makeText((Context)this$0, (CharSequence)"World", 0).show();
      }
    });
  }
}
```

可以看出 `Kotlin Android Extensions` 为每一个 `Activity` 生成了一个名为 `_$_findViewCache` 的 `HashMap` 用于缓存 View id，每次通过 `kotlinx.android.synthetic.*.*.*` 获取 View 其实都是从 Cache 中获取。

* 优点是书写方便。
* 缺点是其实真正执行的代码反而变多了，并且灵活性下降。
* 性能上，由于使用了缓存，所以基本上无差。

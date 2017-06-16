---
layout: post
date: 2017-04-03 00:26:21 +0800
title: Android 中 support-annotations library 整理
tags:
    - Android
---

Copy from 'com.android.support:support-annotations:25.3.1'.

只是整理归纳，用法在注释中写的很清楚了。

## Res

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be an animator resource reference (e.g. {@code android.R.animator.fade_in}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface AnimatorRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be an anim resource reference (e.g. {@code android.R.anim.fade_in}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface AnimRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a resource reference of any type. If the specific type is known, use
 * one of the more specific annotations instead, such as {@link StringRes} or
 * {@link DrawableRes}.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface AnyRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be an array resource reference (e.g. {@code android.R.array.phoneTypes}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface ArrayRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be an attribute reference (e.g. {@code android.R.attr.action}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface AttrRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a boolean resource reference.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface BoolRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a color resource reference (e.g. {@code android.R.color.black}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface ColorRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a dimension resource reference (e.g. {@code android.R.dimen.app_icon_size}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface DimenRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a drawable resource reference (e.g. {@code android.R.attr.alertDialogIcon}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface DrawableRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a fraction resource reference.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface FractionRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be an id resource reference (e.g. {@code android.R.id.copy}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface IdRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be an integer resource reference (e.g. {@code android.R.integer.config_shortAnimTime}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface IntegerRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be an interpolator resource reference (e.g. {@code android.R.interpolator.cycle}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface InterpolatorRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a layout resource reference (e.g. {@code android.R.layout.list_content}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface LayoutRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a menu resource reference.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface MenuRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a plurals resource reference.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface PluralsRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a raw resource reference.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface RawRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a String resource reference (e.g. {@code android.R.string.ok}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface StringRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a styleable resource reference (e.g. {@code android.R.styleable.TextView_text}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface StyleableRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a style resource reference (e.g. {@code android.R.style.TextAppearance}).
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface StyleRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be a transition resource reference.
 */
@Documented
@Retention(SOURCE)
@Target({METHOD, PARAMETER, FIELD})
public @interface TransitionRes {
}
```

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to be an XML resource reference.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
public @interface XmlRes {
}
```

## Thread

``` java
/**
 * Denotes that the annotated method can be called from any thread (e.g. it is "thread safe".)
 * If the annotated element is a class, then all methods in the class can be called
 * from any thread.
 * <p>
 * The main purpose of this method is to indicate that you believe a method can be called
 * from any thread; static tools can then check that nothing you call from within this method
 * or class have more strict threading requirements.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;AnyThread
 *  public void deliverResult(D data) { ... }
 * </code></pre>
 */
@Documented
@Retention(CLASS)
@Target({METHOD,CONSTRUCTOR,TYPE})
public @interface AnyThread {
}
```

``` java
/**
 * Denotes that the annotated method should only be called on the binder thread.
 * If the annotated element is a class, then all methods in the class should be called
 * on the binder thread.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;BinderThread
 *  public BeamShareData createBeamShareData() { ... }
 * </code></pre>
 */
@Documented
@Retention(CLASS)
@Target({METHOD,CONSTRUCTOR,TYPE})
public @interface BinderThread {
}
```

``` java
/**
 * Denotes that the annotated method should only be called on the main thread.
 * If the annotated element is a class, then all methods in the class should be called
 * on the main thread.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;MainThread
 *  public void deliverResult(D data) { ... }
 * </code></pre>
 */
@Documented
@Retention(CLASS)
@Target({METHOD,CONSTRUCTOR,TYPE})
public @interface MainThread {
}
```

``` java
/**
 * Denotes that the annotated method or constructor should only be called on the UI thread.
 * If the annotated element is a class, then all methods in the class should be called
 * on the UI thread.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;UiThread
 *
 *  public abstract void setText(@NonNull String text) { ... }
 * </code></pre>
 */
@Documented
@Retention(CLASS)
@Target({METHOD,CONSTRUCTOR,TYPE})
public @interface UiThread {
}
```

``` java
/**
 * Denotes that the annotated method should only be called on a worker thread.
 * If the annotated element is a class, then all methods in the class should be called
 * on a worker thread.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;WorkerThread
 *  protected abstract FilterResults performFiltering(CharSequence constraint);
 * </code></pre>
 */
@Documented
@Retention(CLASS)
@Target({METHOD,CONSTRUCTOR,TYPE})
public @interface WorkerThread {
}
```

## CallSuper

``` java
/**
 * Denotes that any overriding methods should invoke this method as well.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;CallSuper
 *  public abstract void onFocusLost();
 * </code></pre>
 */
@Documented
@Retention(CLASS)
@Target({METHOD})
public @interface CallSuper {
}
```

## CheckResult

``` java
/**
 * Denotes that the annotated method returns a result that it typically is
 * an error to ignore. This is usually used for methods that have no side effect,
 * so calling it without actually looking at the result usually means the developer
 * has misunderstood what the method does.
 * <p>
 * Example:
 * <pre>{@code
 *  public @CheckResult String trim(String s) { return s.trim(); }
 *  ...
 *  s.trim(); // this is probably an error
 *  s = s.trim(); // ok
 * }</pre>
 */
@Documented
@Retention(CLASS)
@Target({METHOD})
public @interface CheckResult {
    /** Defines the name of the suggested method to use instead, if applicable (using
     * the same signature format as javadoc.) If there is more than one possibility,
     * list them all separated by commas.
     * <p>
     * For example, ProcessBuilder has a method named {@code redirectErrorStream()}
     * which sounds like it might redirect the error stream. It does not. It's just
     * a getter which returns whether the process builder will redirect the error stream,
     * and to actually set it, you must call {@code redirectErrorStream(boolean)}.
     * In that case, the method should be defined like this:
     * <pre>
     *  &#64;CheckResult(suggest="#redirectErrorStream(boolean)")
     *  public boolean redirectErrorStream() { ... }
     * </pre>
     */
    String suggest() default "";
}
```

## ColorInt

``` java
/**
 * Denotes that the annotated element represents a packed color
 * int, {@code AARRGGBB}. If applied to an int array, every element
 * in the array represents a color integer.
 * <p>
 * Example:
 * <pre>{@code
 *  public abstract void setTextColor(@ColorInt int color);
 * }</pre>
 */
@Retention(CLASS)
@Target({PARAMETER,METHOD,LOCAL_VARIABLE,FIELD})
public @interface ColorInt {
}
```

## Dimension

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to represent a dimension.
 */
@Documented
@Retention(CLASS)
@Target({METHOD,PARAMETER,FIELD,LOCAL_VARIABLE,ANNOTATION_TYPE})
public @interface Dimension {
    @DimensionUnit
    int unit() default PX;

    int DP = 0;
    int PX = 1;
    int SP = 2;
}
```

## Range

``` java
/**
 * Denotes that the annotated element should be a float or double in the given range
 * <p>
 * Example:
 * <pre><code>
 *  &#64;FloatRange(from=0.0,to=1.0)
 *  public float getAlpha() {
 *      ...
 *  }
 * </code></pre>
 */
@Retention(CLASS)
@Target({METHOD,PARAMETER,FIELD,LOCAL_VARIABLE,ANNOTATION_TYPE})
public @interface FloatRange {
    /** Smallest value. Whether it is inclusive or not is determined
     * by {@link #fromInclusive} */
    double from() default Double.NEGATIVE_INFINITY;
    /** Largest value. Whether it is inclusive or not is determined
     * by {@link #toInclusive} */
    double to() default Double.POSITIVE_INFINITY;

    /** Whether the from value is included in the range */
    boolean fromInclusive() default true;

    /** Whether the to value is included in the range */
    boolean toInclusive() default true;
}
```

``` java
/**
 * Denotes that the annotated element should be an int or long in the given range
 * <p>
 * Example:
 * <pre><code>
 *  &#64;IntRange(from=0,to=255)
 *  public int getAlpha() {
 *      ...
 *  }
 * </code></pre>
 */
@Retention(CLASS)
@Target({METHOD,PARAMETER,FIELD,LOCAL_VARIABLE,ANNOTATION_TYPE})
public @interface IntRange {
    /** Smallest value, inclusive */
    long from() default Long.MIN_VALUE;
    /** Largest value, inclusive */
    long to() default Long.MAX_VALUE;
}
```

## Def

``` java
/**
 * Denotes that the annotated element of integer type, represents
 * a logical type and that its value should be one of the explicitly
 * named constants. If the IntDef#flag() attribute is set to true,
 * multiple constants can be combined.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;Retention(SOURCE)
 *  &#64;IntDef({NAVIGATION_MODE_STANDARD, NAVIGATION_MODE_LIST, NAVIGATION_MODE_TABS})
 *  public @interface NavigationMode {}
 *  public static final int NAVIGATION_MODE_STANDARD = 0;
 *  public static final int NAVIGATION_MODE_LIST = 1;
 *  public static final int NAVIGATION_MODE_TABS = 2;
 *  ...
 *  public abstract void setNavigationMode(@NavigationMode int mode);
 *  &#64;NavigationMode
 *  public abstract int getNavigationMode();
 * </code></pre>
 * For a flag, set the flag attribute:
 * <pre><code>
 *  &#64;IntDef(
 *      flag = true,
 *      value = {NAVIGATION_MODE_STANDARD, NAVIGATION_MODE_LIST, NAVIGATION_MODE_TABS})
 * </code></pre>
 */
@Retention(SOURCE)
@Target({ANNOTATION_TYPE})
public @interface IntDef {
    /** Defines the allowed constants for this element */
    long[] value() default {};

    /** Defines whether the constants can be used as a flag, or just as an enum (the default) */
    boolean flag() default false;
}
```

``` java
/**
 * Denotes that the annotated String element, represents a logical
 * type and that its value should be one of the explicitly named constants.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;Retention(SOURCE)
 *  &#64;StringDef({
 *     POWER_SERVICE,
 *     WINDOW_SERVICE,
 *     LAYOUT_INFLATER_SERVICE
 *  })
 *  public @interface ServiceName {}
 *  public static final String POWER_SERVICE = "power";
 *  public static final String WINDOW_SERVICE = "window";
 *  public static final String LAYOUT_INFLATER_SERVICE = "layout_inflater";
 *  ...
 *  public abstract Object getSystemService(@ServiceName String name);
 * </code></pre>
 */
@Retention(SOURCE)
@Target({ANNOTATION_TYPE})
public @interface StringDef {
    /** Defines the allowed constants for this element */
    String[] value() default {};
}
```

## Keep

``` java
/**
 * Denotes that the annotated element should not be removed when
 * the code is minified at build time. This is typically used
 * on methods and classes that are accessed only via reflection
 * so a compiler may think that the code is unused.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;Keep
 *  public void foo() {
 *      ...
 *  }
 * </code></pre>
 */
@Retention(CLASS)
@Target({PACKAGE,TYPE,ANNOTATION_TYPE,CONSTRUCTOR,METHOD,FIELD})
public @interface Keep {
}
```

## Null

``` java
/**
 * Denotes that a parameter, field or method return value can never be null.
 * <p>
 * This is a marker annotation and it has no specific attributes.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, ANNOTATION_TYPE, PACKAGE})
public @interface NonNull {
}
```

``` java
/**
 * Denotes that a parameter, field or method return value can be null.
 * <p>
 * When decorating a method call parameter, this denotes that the parameter can
 * legitimately be null and the method will gracefully deal with it. Typically
 * used on optional parameters.
 * <p>
 * When decorating a method, this denotes the method might legitimately return
 * null.
 * <p>
 * This is a marker annotation and it has no specific attributes.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, ANNOTATION_TYPE, PACKAGE})
public @interface Nullable {
}
```

## Px

``` java
/**
 * Denotes that an integer parameter, field or method return value is expected
 * to represent a pixel dimension.
 */
@Documented
@Retention(CLASS)
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE})
@Dimension(unit = Dimension.PX)
public @interface Px {
}
```

## Requires

``` java
/**
 * Denotes that the annotated element should only be called on the given API level
 * or higher.
 * <p>
 * This is similar in purpose to the older {@code @TargetApi} annotation, but more
 * clearly expresses that this is a requirement on the caller, rather than being
 * used to "suppress" warnings within the method that exceed the {@code minSdkVersion}.
 */
@Retention(CLASS)
@Target({TYPE,METHOD,CONSTRUCTOR,FIELD})
public @interface RequiresApi {
    /**
     * The API level to require. Alias for {@link #api} which allows you to leave out the
     * {@code api=} part.
     */
    @IntRange(from=1)
    int value() default 1;

    /** The API level to require */
    @IntRange(from=1)
    int api() default 1;
}
```

``` java
/**
 * Denotes that the annotated element requires (or may require) one or more permissions.
 * <p>
 * Example of requiring a single permission:
 * <pre><code>
 *   &#64;RequiresPermission(Manifest.permission.SET_WALLPAPER)
 *   public abstract void setWallpaper(Bitmap bitmap) throws IOException;
 *
 *   &#64;RequiresPermission(ACCESS_COARSE_LOCATION)
 *   public abstract Location getLastKnownLocation(String provider);
 * </code></pre>
 * Example of requiring at least one permission from a set:
 * <pre><code>
 *   &#64;RequiresPermission(anyOf = {ACCESS_COARSE_LOCATION, ACCESS_FINE_LOCATION})
 *   public abstract Location getLastKnownLocation(String provider);
 * </code></pre>
 * Example of requiring multiple permissions:
 * <pre><code>
 *   &#64;RequiresPermission(allOf = {ACCESS_COARSE_LOCATION, ACCESS_FINE_LOCATION})
 *   public abstract Location getLastKnownLocation(String provider);
 * </code></pre>
 * Example of requiring separate read and write permissions for a content provider:
 * <pre><code>
 *   &#64;RequiresPermission.Read(@RequiresPermission(READ_HISTORY_BOOKMARKS))
 *   &#64;RequiresPermission.Write(@RequiresPermission(WRITE_HISTORY_BOOKMARKS))
 *   public static final Uri BOOKMARKS_URI = Uri.parse("content://browser/bookmarks");
 * </code></pre>
 * <p>
 * When specified on a parameter, the annotation indicates that the method requires
 * a permission which depends on the value of the parameter. For example, consider
 * {@code android.app.Activity.startActivity(android.content.Intent)}:
 * <pre>{@code
 *   public void startActivity(@RequiresPermission Intent intent) { ... }
 * }</pre>
 * Notice how there are no actual permission names listed in the annotation. The actual
 * permissions required will depend on the particular intent passed in. For example,
 * the code may look like this:
 * <pre>{@code
 *   Intent intent = new Intent(Intent.ACTION_CALL);
 *   startActivity(intent);
 * }</pre>
 * and the actual permission requirement for this particular intent is described on
 * the Intent name itself:
 * <pre><code>
 *   &#64;RequiresPermission(Manifest.permission.CALL_PHONE)
 *   public static final String ACTION_CALL = "android.intent.action.CALL";
 * </code></pre>
 */
@Retention(CLASS)
@Target({ANNOTATION_TYPE,METHOD,CONSTRUCTOR,FIELD,PARAMETER})
public @interface RequiresPermission {
    /**
     * The name of the permission that is required, if precisely one permission
     * is required. If more than one permission is required, specify either
     * {@link #allOf()} or {@link #anyOf()} instead.
     * <p>
     * If specified, {@link #anyOf()} and {@link #allOf()} must both be null.
     */
    String value() default "";

    /**
     * Specifies a list of permission names that are all required.
     * <p>
     * If specified, {@link #anyOf()} and {@link #value()} must both be null.
     */
    String[] allOf() default {};

    /**
     * Specifies a list of permission names where at least one is required
     * <p>
     * If specified, {@link #allOf()} and {@link #value()} must both be null.
     */
    String[] anyOf() default {};

    /**
     * If true, the permission may not be required in all cases (e.g. it may only be
     * enforced on certain platforms, or for certain call parameters, etc.
     */
    boolean conditional() default false;

    /**
     * Specifies that the given permission is required for read operations.
     * <p>
     * When specified on a parameter, the annotation indicates that the method requires
     * a permission which depends on the value of the parameter (and typically
     * the corresponding field passed in will be one of a set of constants which have
     * been annotated with a {@code @RequiresPermission} annotation.)
     */
    @Target({FIELD, METHOD, PARAMETER})
    @interface Read {
        RequiresPermission value() default @RequiresPermission;
    }

    /**
     * Specifies that the given permission is required for write operations.
     * <p>
     * When specified on a parameter, the annotation indicates that the method requires
     * a permission which depends on the value of the parameter (and typically
     * the corresponding field passed in will be one of a set of constants which have
     * been annotated with a {@code @RequiresPermission} annotation.)
     */
    @Target({FIELD, METHOD, PARAMETER})
    @interface Write {
        RequiresPermission value() default @RequiresPermission;
    }
}
```

## RestrictTo

``` java
/**
 * Denotes that the annotated element should only be accessed from within a
 * specific scope (as defined by {@link Scope}).
 * <p>
 * Example of restricting usage within a library (based on gradle group ID):
 * <pre><code>
 *   &#64;RestrictTo(GROUP_ID)
 *   public void resetPaddingToInitialValues() { ...
 * </code></pre>
 * Example of restricting usage to tests:
 * <pre><code>
 *   &#64;RestrictScope(TESTS)
 *   public abstract int getUserId();
 * </code></pre>
 * Example of restricting usage to subclasses:
 * <pre><code>
 *   &#64;RestrictScope(SUBCLASSES)
 *   public void onDrawForeground(Canvas canvas) { ...
 * </code></pre>
 */
@Retention(CLASS)
@Target({ANNOTATION_TYPE,TYPE,METHOD,CONSTRUCTOR,FIELD,PACKAGE})
public @interface RestrictTo {

    /**
     * The scope to which usage should be restricted.
     */
    Scope[] value();

    enum Scope {
        /**
         * Restrict usage to code within the same library (e.g. the same
         * gradle group ID and artifact ID).
         */
        LIBRARY,

        /**
         * Restrict usage to code within the same group of libraries.
         * This corresponds to the gradle group ID.
         */
        LIBRARY_GROUP,

        /**
         * Restrict usage to code within the same group ID (based on gradle
         * group ID). This is an alias for {@link #LIBRARY_GROUP}.
         *
         * @deprecated Use {@link #LIBRARY_GROUP} instead
         */
        @Deprecated
        GROUP_ID,

        /**
         * Restrict usage to tests.
         */
        TESTS,

        /**
         * Restrict usage to subclasses of the enclosing class.
         * <p>
         * <strong>Note:</strong> This scope should not be used to annotate
         * packages.
         */
        SUBCLASSES,
    }
}
```

## Size

``` java
/**
 * Denotes that the annotated element should have a given size or length.
 * Note that "-1" means "unset". Typically used with a parameter or
 * return value of type array or collection.
 * <p>
 * Example:
 * <pre>{@code
 *  public void getLocationInWindow(@Size(2) int[] location) {
 *      ...
 *  }
 * }</pre>
 */
@Retention(CLASS)
@Target({PARAMETER,LOCAL_VARIABLE,METHOD,FIELD,ANNOTATION_TYPE})
public @interface Size {
    /** An exact size (or -1 if not specified) */
    long value() default -1;
    /** A minimum size, inclusive */
    long min() default Long.MIN_VALUE;
    /** A maximum size, inclusive */
    long max() default Long.MAX_VALUE;
    /** The size must be a multiple of this factor */
    long multiple() default 1;
}
```

## VisibleForTesting

``` java
/**
 * Denotes that the class, method or field has its visibility relaxed, so that it is more widely
 * visible than otherwise necessary to make code testable.
 * <p>
 * You can optionally specify what the visibility <b>should</b> have been if not for
 * testing; this allows tools to catch unintended access from within production
 * code.
 * <p>
 * Example:
 * <pre><code>
 *  &#64;VisibleForTesting(otherwise = VisibleForTesting.PROTECTED)
 *  public String printDiagnostics() { ... }
 * </code></pre>
 *
 * If not specified, the intended visibility is assumed to be private.
 */
@Retention(CLASS)
public @interface VisibleForTesting {
    /**
     * The visibility the annotated element would have if it did not need to be made visible for
     * testing.
     */
    @ProductionVisibility
    int otherwise() default PRIVATE;

    /**
     * The annotated element would have "private" visibility
     */
    int PRIVATE = 2; // Happens to be the same as Modifier.PRIVATE

    /**
     * The annotated element would have "package private" visibility
     */
    int PACKAGE_PRIVATE = 3;

    /**
     * The annotated element would have "protected" visibility
     */
    int PROTECTED = 4; // Happens to be the same as Modifier.PROTECTED

    /**
     * The annotated element should never be called from production code, only from tests.
     * <p>
     * This is equivalent to {@code @RestrictTo.Scope.TESTS}.
     */
    int NONE = 5;
}
```

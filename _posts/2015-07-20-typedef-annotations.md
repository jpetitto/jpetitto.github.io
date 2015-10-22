---
layout: post
title: Typedef Annotations in Android
comments: true
permalink: typedef-annotations
---

Java's `enum` type is the standard way for defining a set of related constants. For instance, we can define an `enum` to represent different units of temperature:

{% highlight java %}
public enum TemperatureUnit {
    CELSIUS, FAHRENHEIT, KELVIN
}
{% endhighlight %}

The nice thing about an `enum` is that it's type-safe. Wherever a `TemperatureUnit` value is expected, only one of `CELSIUS`, `FAHRENHEIT` or `KELVIN` may be used:

{% highlight java %}
public void setConversionUnit(TemperatureUnit unit) { ... }

setConversionUnit(TemperatureUnit.CELSIUS);
setConversionUnit("Celsius"); // not allowed; 'String' is not of type 'TemperatureUnit'
{% endhighlight %}

`Enums` possess even more power since they can contain both methods and constructors as part of their definition. This allows them to behave similar to `class` types.

Before the `enum` type was introduced in Java 5, programmers would typically define a set of `int` constants instead. Using our previous example of temperature units, we might define the constants as follows:

{% highlight java %}
public class TemperatureConverter {
    public final static int CELSIUS = 1;
    public final static int FAHRENHEIT = 2;
    public final static int KELVIN = 3;
    ...
}
{% endhighlight %}

You'll often find `int` constants used in favor of an `enum` in the Android APIs. For example, the [ViewGroup.LayoutParams](http://developer.android.com/reference/android/view/ViewGroup.LayoutParams.html) class defines the constants `FILL_PARENT`, `MATCH_PARENT` and `WRAP_CONTENT`. The reason for this is related to performance. When compared to `int` constants, [enums may use more than twice the amount of memory](https://developer.android.com/training/articles/memory.html#Overhead). Since many Android devices are resource constrained, memory footprint is a legitimate concern.

The main drawback of using `int` constants though is the lack of type safety. We no longer have the compiler enforcing a value to be from our set of constants. If the `setConversionUnit` method is modified to accept an argument of type `int`, then any integer value can be passed in:

{% highlight java %}
public void setConversionUnit(int temperatureUnit) { ... }

setConversionUnit(CELSIUS);
setConversionUnit(34); // also allowed, but not an expected value
{% endhighlight %}

The Android team at Google [recently introduced](http://tools.android.com/tech-docs/support-annotations) a support library for annotations. Among the many included annotations is the new `@IntDef` annotation. Its purpose is to provide type safety for traditional `int` constants:

{% highlight java %}
@IntDef({CELSIUS, FAHRENHEIT, KELVIN})
@Retention(RetentionPolicy.SOURCE)
public @interface TemperatureUnit {}

// same as before
public final static int CELSIUS = 1;
public final static int FAHRENHEIT = 2;
public final static int KELVIN = 3;
{% endhighlight %}

We define an annotation named `TemperatureUnit`, acting as a namespace for our set of constants. We apply the `@Retention` annotation to its definition with a value of `RetentionPolicy.SOURCE` since we only need it to exist at compile time. Finally, we add the `@IntDef` annotation with the names of our constants.

Now we can annotate our code with `@TemperatureUnit` and get the added type safety:

{% highlight java %}
public void setConversionUnit(@TemperatureUnit int temperatureUnit) { ... }

setConversionUnit(CELSIUS);
setConversionUnit(34); // no longer allowed
{% endhighlight %}

The compiler will generate a warning in the case that one of expected constant values is not provided. While not as forceful as a compile-time error, it can be helpful in notifying both developers and continuous integration servers of type safety issues.

Sometimes it makes sense to apply more than one constant value at a time. For this, the `@IntDef` annotation provides a `flag` attribute, allowing multiple constant values to be combined using bitwise operators:

{% highlight java %}
@IntDef(flag=true, values={
    BOLD, ITALIC, UNDERLINE
})
@Retention(RetentionPolicy.SOURCE)
public @interface TextStyle {}

public final static int BOLD = 1;
public final static int ITALIC = 2;
public final static int UNDERLINE = 3;

public void setTextStyle(@TextStyle int style) { ... }

setTextStyle(BOLD|ITALIC);
{% endhighlight %}

Now multiple constant values can be used simultaneously. When a value of combined constants is passed, the pattern is type-checked by the `@IntDef` annotation. An alternative is to use the [EnumSet](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html) class from the Java APIs. The issue again is the memory consumption when compared to the more lightweight `int` constants.

The other typedef annotation that the support library provides is `@StringDef`. While having `int` constants is sufficient in most cases, treating constant values as `Strings` can be useful. We recently made use of this in [CafeJava](https://github.com/IBM-MIL/CafeJava), which provides [RxJava](https://github.com/ReactiveX/RxJava) extensions to the [MobileFirst Platform](http://www-03.ibm.com/software/products/en/mobilefirstplatform) API. The API allows the programmer to specify an HTTP method for a `WLResourceRequest`. It expects a `String` and within that class is a set of `String` constants representing the different HTTP methods. CafeJava takes advantage of the `@StringDef` annotation to ensure a valid constant value is passed in:

{% highlight java %}
@StringDef({GET, POST, PUT, DELETE})
@Retention(RetentionPolicy.SOURCE)
public @interface HttpMethod {}

public static final String GET = WLResourceRequest.GET;
public static final String POST = WLResourceRequest.POST;
public static final String PUT = WLResourceRequest.PUT;
public static final String DELETE = WLResoureRequest.DELETE;
{% endhighlight %}

Thanks to these new annotations, we now have a way to specify sets of constants that are both performant and type-safe. You can start using them in your own code by adding the following gradle dependency:

{% highlight groovy %}
compile 'com.android.support:support-annotations:22.1.1'
{% endhighlight %}

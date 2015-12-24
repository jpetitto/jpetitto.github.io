---
layout: post
title: Dependency Management for Android Libraries
comments: true
permalink: android-lib-dependency-management
---

<!-- excerpt.start -->When Android developers choose a library for their project, they aren't merely looking for things such as features, usability, performance, documentation and support. They also care how large the library is and the number of methods it's going to add. As a project grows and so does its dependencies, developers feel the pressure to keep their app below the 65k method limit. Proguard is too slow to wait for with non-release builds and developers try to avoid multidex like the plague. It's therefore crucial that library authors be concious about the size of their projects.

The simplest approach to keeping your library's method count down is to not include any unnecessary dependencies. Any dependencies you do include will transitively be added to your users' projects. For example, if you need a few simple utility methods, such as closing a resource quietly, don't go adding [Guava](https://github.com/google/guava) just to do this. Instead, roll your own or extract it from an existing library (make sure to give credit!). Your users will most definitely appreciate the exclusion of over 14k methods when you only needed a few.<!-- excerpt.end -->

That's not to say you should always avoid using external libraries; you just need to be smart about it. Don't go out of your way to write an HTTP client when such libraries already exist. You'll just end up wasting time that could be better spent improving your own library.

Beyond simply deciding which libraries to include, there are a few strategies you can employ to help keep your libary lean. One such strategy is to use the `provided` scope when declaring dependencies. This is part of the Android build system in gradle. As opposed to `compile`, the `provided` scope only includes a dependency at compile time. This means that the dependency will not be packaged with the APK when users build their projects. Users will need to explicitly declare the dependency themselves in their app's `build.gradle` for it to be available at runtime.

*Note: there is a also a `package` scope which does the opposite of `provided`. It gets packaged with the APK but is not available at compile time.*

There are a few reasons why you would want to use an optional dependency in your library. One reason is that certain features may only get used by a subset of your users. An example of this can be seen with [Retrofit 1.x](https://github.com/square/retrofit/tree/version-one), where it's possible to consume REST calls reactively instead of using callbacks. Those that want to use RxJava can add it and those that don't aren't burdened by the extra dependency. The configuration for this is slightly different with Retrofit since it uses the [maven build system](https://github.com/square/retrofit/blob/version-one/retrofit/pom.xml#L35), but the ideas are similar.

I should warn that if you find yourself including features that not all users may find useful, you should really consider if those features should even be part of your library. More on this a bit later.

Another reason you might want to include an optional dependency is when a solution already exists in the Android framework, but a more performant solution is available from an external library. Users that already rely on this library or are willing to take on the increased method count for better performance can add it.

I recently came across this in the [PlacesAutocompleteTextView library](https://github.com/seatgeek/android-PlacesAutocompleteTextView), where the internal HTTP client used can either be an `OkHttpClient` or an `HttpURLConnection`. The former is generally more performant, but requires adding [OkHttp](http://square.github.io/okhttp/) as a dependency. If users do not wish to include it, then it automatically falls back to `HttpURLConnection` from the standard library.

To achieve this, a "resolver" class is used to determine which dependency to use at runtime. For example, this is the class for determining which HTTP client to use:

{% highlight java %}
public final class PlacesHttpClientResolver {
  public static final PlacesHttpClient PLACES_HTTP_CLIENT;

  static {
    boolean hasOkHttp;
    
    try {
      Class.forName("com.squareup.okhttp.OkHttpClient");
      hasOkHttp = true;
    } catch (ClassNotFoundException e) {
      hasOkHttp = false;
    }

    PlacesApiJsonParser parser = JsonParserResolver.JSON_PARSER;

    PLACES_HTTP_CLIENT = hasOkHttp ? new OkHttpPlacesHttpClient(parser) : new HttpUrlConnectionMapsHttpClient(parser);
  }

  private PlacesHttpClientResolver() {
    throw new RuntimeException("No Instances!");
  }
}
{% endhighlight %}

When the class gets loaded, the fully qualified class name for `OkHttpClient` is checked for availability. If a `ClassNotFoundException` is thrown, then we know OkHttp wasn't added by the user and we fall back to `HttpURLConnection`. `PlacesHttpClient` acts as common interface that wraps both implementations so either can be used interchangeably through out the codebase. This same approach is used for JSON parsing, where [Gson](https://github.com/google/gson) can optionally be included as a dependency.

This approach is good if the tradeoff between performance and size is significant. In the case where the fallback implementation requires greater effort to use (such as with JSON parsing), I'd recommend going with the external library first to help save time and then consider adding the fallback implementation in a later release.

I mentioned earlier that you should be wise about which features to include in your library. If a feature isn't going to be needed by nearly all of your users, then it's probably best not to include it. This makes the first approach of using an optional dependency much less warrented. Revisiting Retrofit as an example, it no longer provides the ability to consume REST calls reactively as part of its core library in the [2.x release](http://square.github.io/retrofit/). Instead, this functionality has been moved to a separate module and published as its own maven artifact.

Likewise, different response converters have been split into their own dependencies as well. For example, Retrofit users that need to convert a JSON response and already rely on Gson can add the following dependency to their `build.gradle` file:

{% highlight groovy %}
dependencies {
  compile 'com.squareup.retrofit:converter-gson:2.0.0-beta2'
}
{% endhighlight %}

Those that use a different JSON library like [Jackson](http://wiki.fasterxml.com/JacksonHome) or need to parse a different data format such as XML or protocol buffers, can do so and not be burdened by the extra dependencies that would be needed to serve all of Retrofit's users. Just as important, the core library isn't muddied up by these additional features and can remain focused on the primary issues it's meant to solve.

If you find yourself writing a library that Android deverlopers will use, keep some of these strategies in mind when designing it. Consider the size of your library not merely as an attribute, but as a feature. Your users will certainly appreciate you for it!

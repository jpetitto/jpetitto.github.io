---
layout: post
title: Loading Resources from Unit Tests
comments: true
permalink: unit-test-resource-loading
---

<!-- excerpt.start -->There are a handful of reasons why you'd want to load in a resource file for a test:

* Input (or output) data is large and complex
* Input (or output) data is not easily mockable, such as a `File` or `InputStream`
* Configuration data is preferably externalized

One option would be to place your resources in your app's `/res` or `/assets` directory. This requires instrumenting your tests so that you can access the resources through the app's `Context`. This can be undesirable for two reasons:<!-- excerpt.end -->

1. If your tests do not require any additional instrumentation, then you're sacrificing the speediness of running tests on a JVM.
2. You're testing against a library and therefore don't have an app to instrument.

Accessing resources from plain-old JUnit tests turns out to be fairly straightforward. Place any resources into a new directory named `/src/test/resources` and then access them through the test's `ClassLoader`:

{% highlight java %}
ClassLoader classLoader = getClass().getClassLoader();
URL resource = classLoader.getResource("data.json");
File file = new File(resource.getPath());
{% endhighlight %}

This method may not work for older versions of Android Studio or the Android Gradle plugin. Instead you have to explicitly copy the resource files into the module's build directory. This can be done by writing a task in the `build.gradle` file:

{% highlight groovy %}
task copyResourcesToClasses(type: Copy) {
  from "${projectDir}/src/test/resources"
  into "${buildDir}/intermediates/classes/test/debug/resources"
}
{% endhighlight %}

You will need to execute the task before running your tests. Moreover, the file name passed to the `ClassLoader` will need to be prefixed with the resources directory (e.g. `resources/data.json`). Since you're explicitly copying the files over in this case, the directory name is arbitrary.

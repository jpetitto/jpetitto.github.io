---
layout: post
title: The Silliness of Nullability and Views
comments: true
permalink: api-null-annotations
---

<!-- excerpt.start -->
Since Google introduced the support annotations library, there's been an increase in the application of the `@Nullable` and `@NonNull` annotations in APIs. It's helpful in languages such as Java where optional values are not first class citizens of the language. Static analyzers can infer at compile time if an object has the possibility of being null when passed to (or returned from) a method.

While the benefits of these annotations can certainly be helpful, they may also hinder users of an API. A great case in point would be that of the `findViewById` method in Android's `ActivityAppCompat` class:<!-- excerpt.end -->

{% highlight java %}
@Nullable
public View findViewById(@IdRes int id) {
    ...
}
{% endhighlight %}

It's perfectly fine for this method to return null when we supply an invalid `id`. But the added `@Nullable` on the return value makes this method annoying to use:

{% highlight java %}
View someView = findViewById(R.id.some_view);
if (someView != null) {
	someView.setVisibility(View.GONE);
}
{% endhighlight %}

Due to the annotation, we must first check that the `View` returned is not null before proceeding to use it.

Even though the value returned by `findViewById` could be null, we don't ever expect it to be at runtime. In fact, if the view was null because we used the wrong id or the wrong view, we would want our app to crash so we could fix it immediately in our code. [Jesse Wilson wrote about this in an article on his blog.](https://publicobject.com/2015/06/07/null/).

If we were to use an `Optional` value instead, it would be perfectly suitable to use the `get` operation in this case. Even though it would throw a runtime exception if the value was null, we know it never would be. iOS developers are familiar with this in Swift with using the `!` operator to force unwrap a `UIView` that has been connected by an `@IBOutlet`.

Curiously enough, `findViewById` in the `View` class does not use the `@Nullable` annotation, making the more typical approach of finding views (e.g. in Fragments) simpler to use.

So if you're writing APIs, decide carefully if your users need to deal with null values or not. In some cases it might not make sense.
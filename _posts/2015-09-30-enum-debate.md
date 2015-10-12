---
layout: post
title: The Great Enum Debate
---

*This article was originally published on [IBM's Mobile Innovation Lab blog](https://ibm.biz/enum-debate)*

-----

Since the early days of Android, there has been an ongoing debate about whether or not enums should be used due to their overhead in memory consumption. Google has long been in favor of using int constants as a substitute and even go as far as saying [enums should be strictly avoided](https://developer.android.com/training/articles/memory.html#Overhead). A memory comparison between the two methods is neatly outlined in [this Stack Overflow answer](http://stackoverflow.com/a/25306325).

Recently, this debate has come back into the spotlight. One reason for this is that Google has been pushing the issue lately as part of their [#perfmatters](https://twitter.com/hashtag/perfmatters) campaign, which educates Android developers on performance related topics. They've also released a new [support annotations library](http://tools.android.com/tech-docs/support-annotations) containing the @IntDef annotation, which provides compile-time type checking for int constants (I wrote an article on it [here](https://github.com/jpetitto/blog-articles/tree/master/android-typedef-annotations)). As a result of all of this, there has been [backlash](https://twitter.com/parallelcross/status/636982705154949120) from some prominent community members regarding the merit of Google's stance on enums.

[Colt McAnlis](https://twitter.com/duhroach), a developer advocate at Google, has been leading the charge against enums. His argument stems from the fact that memory is a shared and precious resource on Android devices and it's the developer's responsbility to minimize the impact their app has on the rest of the system. With that said, I don't think eliminating the use of enums alone are going to make enough of a difference for the end user to even notice.

That's not to say the optimization isn't warranted though. [Jake Wharton](https://twitter.com/JakeWharton) recently gave a talk [discussing different micro optimizations](https://www.youtube.com/watch?v=b6zKBZcg5fk) that you can apply to your code base. One of the points he made was that a collection of micro optimizations together may often outweigh the impact that a single macro optimization can have on performance. It surprises me then that he would oppose the use of the @IntDef annotation in his [ironic, but humorous tweet](https://twitter.com/JakeWharton/status/634591335815729152). I would think that avoiding enums falls under the micro optimization category.

McAnlis also wrote about the idea of [preventative versus premature optimizations](https://medium.com/google-developers/the-truth-about-preventative-optimizations-ccebadfd3eb5). Avoiding enums are much more of a preventative strategy than anything; you're not going out of your away to implement the optimization. It's a simple choice between two different methods of implementation that will yield similar results in terms of semantics in your code.

In [Bob Lee's](https://twitter.com/crazybob) recent rant about how [avoiding enums is unjust](https://twitter.com/parallelcross/status/636982705154949120), he makes the point that enums and int constants don't necessarily aim to solve the same problem, as their use cases are not uniform. While this is certainly true, enums are often used in their simplest capacity, which is to define a set of related constant values. If you don't need all the power of an enum and the values of the constants do not matter (besides being unique), then the idea of using int constants over enums becomes more justified in my view.

At the end of the day, enums alone won't make or break the performance of your app. There are times when you almost always want to use them, such as when writing an API that will be consumed by other developers. But when you're writing code for an app, you typically have greater flexibility in the decisions you make. With the added benefits that the @IntDef annotation now provides, using int constants over enums isn't nearly as dangerous as it used to be. So why not favor the one that will end up saving you a little bit of memory? Because when it's all said and done, it's a small step towards ensuring your users have the best experience possible when using their device.

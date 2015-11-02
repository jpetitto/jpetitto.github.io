---
layout: post
title: A hard-to-catch bug... and understanding UIAppearance
comments: true
permalink: understanding-uiappearance
---

*This article was originally published on [IBM's Mobile Innovation Lab blog](http://www-969.ibm.com/innovation/milab/blog/post/a-hard-to-catch-bug-and-understanding-uiappearance)*

-----

<!-- excerpt.start -->My first encounter with the `UIAppearance` protocol came after many frustrating hours of hunting down an inexplicable bug. In a recent project here at MIL, every `UISegmentedControl` object in our app had its appearance altered after segueing to a specific view controller. New to the iOS platform and oblivious that such a protocol existed, I had no clue why such behavior was occurring.

I eventually stumbled across the following line of code in the aforementioned view controller:

{% highlight objc %}
[[UISegmentedControl appearance] setWidth:110.0f forSegmentAtIndex:1];
{% endhighlight %}

A quick web search returned several results describing the `UIAppearance` protocol and its ability to customize UI components across a single app. Clearly, this line of code was unintended since we did not want the second segment of every `UISegmentedControl` to have a fixed width.<!-- excerpt.end -->

While the previous case shows the danger and possible misuse of the `UIAppearance` protocol, it does introduce a powerful and useful concept that allows developers to easily control the look-and-feel of UI components across an entire app. The rest of this article delves into the details of using the protocol.

Several `UIView` based classes (and a few others too) conform to the `UIAppearance` protocol. This allows you to fetch the appearance proxy from one of these classes by calling its `appearance` method. For the `UIView` class, this is accomplished with the following syntax:

{% highlight objc %}
[UIView appearance];
{% endhighlight %}

This proxy enables you to call methods and set properties on the receiver class (`UIView` in this case) that are marked with the `UI_APPEARANCE_SELECTOR` macro. [Here is a list](https://gist.github.com/mattt/5135521) of classes that currently conform to the `UIAppearance protocol and the corresponding properties and methods they support.

You'll notice only a handful of properties and methods in each class are available to use via the appearance proxy. You may have also noticed that the `setWidth:forSegmentAtIndex` method that was called on the `UISegmentedControl` class is not listed. It turns out that some methods simply work with the appearance proxy despite not being marked with the `UI_APPEARANCE_SELECTOR` macro. In such cases, it's best to avoid calling these methods as they may not work as intended in future updates to the iOS platform.

While the appearance proxy method is the most straightforward way of applying UI changes to a class across all of its instances, the protocol offers additional methods for restricting which instances are targeted. The most common method for doing this is `appearanceWhenContainedIn:`, which allows you to specify a class containment hierarchy for the appearance proxy to be applied to. The classes you specify in the containment hierarchy must conform to the `UIAppearanceContainer` protocol, which includes `UIView`, `UIPopoverController`, `UIViewController` and their subclasses.

It's important to note that the order in which the classes of the containment hierarchy are listed influences which instances of a particular class are modified. For example, if you only want to change the title color of buttons that are inside a table view of a specific view controller, you would call the `appearanceWhenContainedIn:` method like so:

{% highlight objc %}
[[UIButton appearanceWhenContainedIn:[ExampleViewController class], [UITableView class], nil] setTitleColor:[UIColor orangeColor] forState:UIControlStateNormal];
{% endhighlight %}

If you reversed the order of the specified container classes, then the `setTitleColor:forState:` method would be applied only to instances of `UIButton` that are contained within an instance of `ExampleViewController`, which in turn is contained within a `UITableView`. This is probably not what you intended when calling `appearanceWhenContainedIn:`.

You can further refine which instances of a class are targeted by specifying a `UITraitCollection` with the `appearanceForTraitCollection:` method. A trait collection describes a set of UI characteristics for a view controller, such as size and scale. This can be used in conjunction with a containment hierarchy via the `appearanceForTraitCollection:whenContainedIn:` method.

Overall, the `UIAppearance` protocol can be an effective means for applying uniform UI changes across an entire application or a specific subset. Meanwhile, it can introduce unexpected behavior when used in a code base that is shared among multiple developers. Unless your design specifies a style requirement to be applied across the entire app, it is best to use the appearance proxy within the most restrictive containment hierarchy deemed useful.

**Additional Resources:**

If you are implementing a custom view class and want to be able to support the `UIAppearance` protocol for certain properties and methods of your class, [read this blog article](http://petersteinberger.com/blog/2013/uiappearance-for-custom-views/) by Peter Steinberger on doing so.

[UIAppearance Protocol Reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIAppearance_Protocol/index.html)
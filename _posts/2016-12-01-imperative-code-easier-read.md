---
layout: post
title: Is Imperative Code Easier to Read?
comments: true
permalink: imperative-code-easier-read
---

<!-- excerpt.start -->
[Rob Napier](https://twitter.com/cocoaphony) recently gave a gentle introduction to [functional programming in Swift](https://realm.io/news/tryswift-rob-napier-swift-legacy-functional-programming/). Even for experienced functional programmers it's a short but enjoyable talk to watch. The talk was trending on Hacker News a few weeks back and there was some [interesting discussion](https://news.ycombinator.com/item?id=13006459) around functional code not always being easier to read than its imperative counterpart, albeit usually being shorter.<!-- excerpt.end -->

Below is one of the examples from the talk:

{% highlight swift %}
let persons = names
    .map(Person.init)
    .filter { $0.isValid }
{% endhighlight %}

and its analogous imperative version:

{% highlight swift %}
var persons: [Person] = []
for name in names {
    let person = Person(name: name)
    if person.isValid {
        persons.append.(person)
    }
}
{% endhighlight %}

This type of example is common in many introductions on the subject and often the same conclusion is jumped too without much consideration: it's easier to read.

I think most (all?) programmers used to writing code in a functional manner will agree that it's easier to follow the first example (not to mention fewer places for bugs to hide). Yet for someone used to writing code in an imperative style, it will most likely be harder to read. The approach is foreign to them and words like `map` and `filter` and a magic `$0` probably don't make a whole lot of sense.

Likewise someone can be equally perplexed by the use of looping constructs if it's the first time they're seeing such a thing. It may be hard to think back to the first time you saw a `for` loop, but it was probably a bit strange until you started to become more comfortable with it. This is no different than learning what a function like `map` does and how to use it mechanically.

To be fair, Swift helps improve readability by using a `for-in` loop instead of the more traditional style `for` loop. Seeing such a loop (as in Java or C) would be even more strange at first:

{% highlight java %}
for (int i = 0; i < names.length; i++) {
    ...
}
{% endhighlight %}

A `map` over a `for` loop isn't much of an improvement on its own. The real benefit comes as the code grows and the solution becomes increasingly more complex. The disparity in size becomes larger and the inherent noise of traditional imperative mechanisms becomes more apparent. The functional solution distills the problem into its most essential pieces and no more. It's not only shorter but also more concise. It might take a little time though before warming up to it.
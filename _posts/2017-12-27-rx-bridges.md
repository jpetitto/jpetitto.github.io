---
layout: post
title: Don't build bridges where there's no water to cross
comments: true
permalink: rx-bridges
---

<!-- excerpt.start -->
At work we've been employing RxJava across our codebase. This means that most methods we write, especially those involving database or network calls, will return an Rx type (e.g. Observable, Single, Completable).

For example, there's an operation to delete a credit card from a user's virtual wallet. This requires sending a request to our server to delete the card. As a side effect, we also want to delete this card locally from our database when the network call completes successfully.
<!-- excerpt.end -->

If you're familiar with RxJava, this code should look straightforward:

{% highlight java %}
public Completable deleteCustomerCard(CustomerCard card) {
    return cardService.deleteCard(card)
            .andThen(Completable.fromAction(() -> database.deleteCard(card)));
}
{% endhighlight %}

In both cases `deleteCard` returns a Completable that will perform the desired operation when subscribed to. The issue though is that the second operation, deleting the card from the database, will never be executed.

As mentioned, `database.deleteCard(card)` returns a Completable. Wrapping it in a second Completable with `Completable.fromAction(...)` is not only unnecessary, it actually prevents the nested Completable from ever executing.

The fact is, `Completable.fromAction(...)` simply executes its Action when it's subscribed to. In this case the action is to call `database.deleteCard(card)`. Since this call simply returns another Completable, the actual work we want done is never subscribed to and executed.

This mistake seems obvious when pointed out, but at the time it was an honest and easy mistake to make. This is especially true when you have developers who are new to the Rx paradigm. Ideally this would be caught in a code review but sometimes things like this slip through the cracks.

If your code is largely designed in a reactive manner, you usually won't need to use static factory methods. This is often left as an implementation detail inside the lower level methods that need to bridge the gap between the Rx and non-Rx worlds.

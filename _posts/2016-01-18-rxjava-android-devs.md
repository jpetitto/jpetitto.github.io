---
layout: post
title: RxJava for Android Developers (Talk)
comments: true
permalink: rxjava-android-devs
---

<!-- excerpt.start -->Last week [I gave a talk on RxJava](http://www.meetup.com/Austin-Android/events/226870556/) at the [Austin Droids meetup](http://www.meetup.com/Austin-Android/). It marked the group's fifth year in existence (congrats to the organizers!) and was also the most attended meetup in its history. I was hoping to have the talk recorded but after some technical issues we weren't able to make it happen. [I posted the slides online](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup) after the talk but I don't think they stand too well on their own. So I've taken the time to write out my narrative to the slides below.

<script async class="speakerdeck-embed" data-id="cc07d5f4584c4e40b4627919b377a582" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

<br />
<!-- excerpt.end -->

[Slide 2](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=2):

As Android developers we do lots of asynchronous programming. The main reason for this is that we don't want to block the main thread, which is responsible for updating our app's UI. If we do, then the app will pause and cause a frustrating experience for our users. We must move any potentially long running operations off of the main thread and into the background until it's done.

[Slide 3-6](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=3):

Async programming is not always easy though. As the complexity of our apps grow, the ability to control its async behavior becomes increasingly difficult.

For instance, composing multiple async calls together is not always easy. Nesting of calls, where each subsequent call relies on the previous one, can become difficult to manage. Or if we have multiple calls that can run simultaneously, then we need a way of ensuring each call has completed before proceeding.

To make these calls asynchronous we have to introduce multiple threads. And I don't think I need to tell anyone just how difficult it can be to manage concurrency in your app. Depending on the abstractions you're using and the complexity of the problem at hand, it may be difficult to prove that your program runs the way you expect it to under all conditions.

On top of this, we have to think about error handling. Each call could perhaps trigger its own set of errors. And depending on the complexity of each call, there may be a lot of boilerplate to ensure each error is caught and handled appropriately. This can make composition even more difficult.

It's these three issues together that can make asynchronous code difficult to manage.

[Slide 7](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=7):

Assuming you aren't using [RxJava](https://github.com/ReactiveX/RxJava) yet, then your Android project probably contains a lot of callbacks. If you're using an existing API, perhaps it takes care of the concurrency and error handling to some extent for you. But you still need to worry about composition.

And when you can't rely on someone else's code to do this, you've probably used something like [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html). There's nothing inherently wrong with an AsyncTask, but it's severely limiting in terms of its capabilities. Composing multiple AsyncTasks together can be painful and if you aren't making a round trip from the main thread to a background thread, then it's not going to suit your needs.

Perhaps you fall back to using [Handlers](http://developer.android.com/reference/android/os/Handler.html) when an AsyncTask doesn't do what you need. Or maybe you even use abstractions from [Java's concurrency API](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/package-summary.html), such as an [Executor](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Executor.html). In a last ditch effort you might even find yourself using the lowest abstraction of all: the [Thread](https://docs.oracle.com/javase/7/docs/api/java/lang/Thread.html).

None of these tools offer us the help we need when dealing with the previously mentioned issues. If only there was something that could help us...

[Slide 8-9](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=8):

This is where RxJava comes into play.

At this point you're probably asking what RxJava actually is. I put the following definition together to try and encapsulate its main ideas:

RxJava is a **library** for producing, **composing** and consuming **asynchronous streams**. It provides powerful abstractions for **concurrency** and makes **error handling** a breeze.

I want to point out that RxJava is a library, not a framework; meaning we can use as little or as much of it as we want in our code. You don't need to commit your app's architecture to a new programming ideology or way of thinking in order to use it. You can simply augment a small piece of your codebase to start reaping the benefits of RxJava. This is especially nice if you already have a large app and slowly want to refactor bits and pieces of it. If you're like me though, you'll probably try and apply RxJava to all parts of your codebase.

I also highlighted the term asynchronous stream. A stream is just a sequence of elements. If you've used or looked at Java 8 before, then you're probably familiar with its [Stream API](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html). RxJava offers something similar except it's non-blocking (asynchronous).

Finally, you'll notice that RxJava is going to help address the three issues of async programming that were outlined before.

[Slide 10-16](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=10):

Now on to some RxJava code. The core type in RxJava is the [`Observable`](http://reactivex.io/documentation/observable.html) type, which represents a producer in our async streaming model.

For the purposes of this talk, I'm going to be using lambda expressions in the example code. These will replace any existing anonymous classes that we might have used and are much more concise to read and write. As Android developers we can actually use a project called [retrolambda](https://github.com/orfjackal/retrolambda) to backport many of Java 8's features to Java 6.

First thing to notice is the static method call [`create`](http://reactivex.io/documentation/operators/create.html). As you can guess, this allows us to create a new `Observable`. Also notice that a [`subscriber`](http://reactivex.io/documentation/operators/subscribe.html) is passed into the method. This represents the consumer, which we'll get to shortly. Basically every time there's a new `subscriber`, the `Observable` will begin producing its values.

We can forward these values to our `subscriber` with its `onNext` method. In this case we are sending it the string value `"Hello"`. This means our `Observable` will emit values of type `String`.

Since we are working with a stream of values, we can go ahead and emit a second value if we'd like. The number of `onNext` notifications our `Observable` generates could potentially be infinite.

When we are done emitting values, we can signal to the `subscriber` that no more values will be emitted. This is done through its `onCompleted` method. It's important to note that this is a terminal event, meaning no more notifications can sent from the `Observable` to its `subscriber`.

A third type of notification is `onError`. This allows us to forward any possible errors to the `subscriber`. Like `onCompleted` though, this is a terminal event and no more notifications can be sent by the `Observable`.

[Slide 17](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=17):

We can actually simplify the creation of our `Observable` with the helper method [`just`](http://reactivex.io/documentation/operators/just.html). It does the same thing we would have done on our own with `create` but automatically calls `onCompleted` for us after each value specified is emitted with `onNext`.

[Slide 18-22](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=17):

There are actually quite a few methods to help us create `Observables`. For example, there is the [`from`](http://reactivex.io/documentation/operators/from.html) method which takes an `Iterable` type, in this case a `List`. It's also overloaded to accept an array as well.

[`concat`](http://reactivex.io/documentation/operators/concat.html) is useful when we have a series of existing `Observables` that we want to subscribe to in succession and emit their values in order.

[`merge`](http://reactivex.io/documentation/operators/merge.html) is similar to `concat` but the `Observables` are subscribed to simultaneously and values emitted by each `Observable` can be interleaved. That's why in the example I can't pass in two separate `Observables` for the strings `"Hello"` and `"again..."`. It's important to keep this in mind when order plays an important role.

[Slide 23](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=23):

We'll be using this `getSupportedVersions` method in the subsequent examples. It simply returns a list of Android versions; in this case Kit-Kat, Lollipop and Marshmallow.

[Slide 24-29](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=24):

As mentioned previously, `Subscribers` are the consumers in our async streams. Notice that we are wrapping the creation of `Observable` with the [`defer`](http://reactivex.io/documentation/operators/defer.html) method. Since the `getSupportedVersions` method would get evaluated when it's passed to the `from` method as an argument, we can prevent this from happening until we actually have a subscriber. This is helpful when the data source for the `Observable` is potentially expensive.

We simply call `subscribe` to subscribe to our `Observable`. At this time it will go and create a new `Observable` from the list of Android versions that get returned by the `getSupportedVersions` method.

We can pass a function to the `subscribe` method which will get called every time a new value is emitted by the `Observable`. In this case we are just printing out the name of each version.

We can also go ahead and pass a second callback for any errors that might get generated by the `Observable`. This is one of the key components to RxJava that makes error handling much easier. Before we would have needed to worry about handling errors at any point where something could go wrong; here it is centralized to the function we pass to `subscribe`. It's worth mentioning that if we don't supply an error callback and the `Observable` does generate one, then RxJava will throw an `Exception` indicating that the error wasn't handled.

Finally, we can supply a callback for when the `Observable` is completed. Notice on the next slide that we could have passed in a `Subscriber` directly instead of three separate callbacks. This is what the other calls boil down to under the hood.

[Slide 30-32](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=30):

Now I'm going introduce operators. Operators allow us to create new `Observables` from existing ones. They are also helpful when it comes to composing multiple `Observables` together.

So if you've done any functional programming before, this will be very familiar. The first operator we'll look at is [`map`](http://reactivex.io/documentation/operators/map.html). What `map` does is take each value emitted by the source `Observable` and transforms it into a new value based on the function passed in. In this case we're calling the `toString` method on each version that gets emitted. It's important to note that `map` returns a new `Observable`. So while the original `Observable` emits `Versions`, the new one after the `map` operator emits `Strings`.

We can continue to chain operators, each one returning a new `Observable`. For example, here is the [`filter`](http://reactivex.io/documentation/operators/filter.html) operator which takes a function and determines if the value should be passed on to the consumer or not. In this case, we are no longer supporting Kit-Kat as a version (sorry!).

[Slide 33](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=33):

Here's another method that we'll be using in our upcoming examples. It takes a version and returns us the names of the devices that run on that particular version.

[Slide 34](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=34):

Let's take a look now at the [`flatMap`](http://reactivex.io/documentation/operators/flatmap.html) operator. It takes each item emitted by the source `Observable` and then returns a new `Observable` for each item. In this case, each `Observable` represents a list of device names for a particular version. `flatMap` will then take each `Observable` that gets returned and "flatten" them into a single `Observable`.

[Slide 35](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=35):

Now `flatMap` uses a `merge` under the hood and as I previously mentioned the `merge` operator interleaves its items. So if order matters, we can swap out `flatMap` for [`concatMap`](http://reactivex.io/RxJava/javadoc/rx/Observable.html#concatMap(rx.functions.Func1)) to preserve ordering. This way all the devices for Kit-Kat will get emitted before the devices for Lollipop (and eventually Marshmallow) do.

[Slide 36](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=36):

There are tons of operators defined for us in RxJava. If there's something you want to do, chances are it's already covered by an operator or some combination of operators from the library. Take a look through the RxJava docs sometime to see what's available.

[Slide 37-39](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=37):

In the rare case you can't find what you're looking for, you can always [write your own operator](http://reactivex.io/documentation/implement-operator.html) that implements the `Operator` interface in `RxJava`. You can then apply it to an existing `Observable` with the `lift` operator.

You can also apply multiple operators with a `Transformer` that you pass to the `compose` operator. This is useful when you don't want to repeat the same set of operations across multiple `Observable` chains.

[Slide 40-42](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=40):

There's something I haven't told you yet, which is that every `Observable` we've seen so far has been synchronous, or blocking. In order to make them asynchronous we are going to use RxJava's [`Schedulers`](http://reactivex.io/documentation/scheduler.html) API.

[Slide 43](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=43):

A `Scheduler` is an abstraction for concurrency and allows us to use multiple threads. `Schedulers` primarily do two things: they determine how something should be scheduled (such as with a thread, thread pool, Executor) and when it should be executed (immediately or at some point in the future).

[Slide 44-54](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=44):

So here's the same `Observable` that we've seen get subscribed to before. We can make it asynchronous by applying the [`subscribeOn`](http://reactivex.io/documentation/operators/subscribeon.html) operator and passing it a `Scheduler`. The `subscribeOn` operator tells the source `Observable` how and when its work should be scheduled when there's a new `Subscriber`.

In this example, we're telling the source `Observable` to do its work on a new thread. RxJava [provides several pre-defined](http://reactivex.io/RxJava/javadoc/rx/schedulers/Schedulers.html) `Schedulers` that we can use: `io` for a pool of background threads, `immediate` for the current thread and `from` if we want to use an `Executor` from Java's concurrency API.

There is also an [`observeOn`](http://reactivex.io/documentation/operators/observeon.html) operator that we can use. This effects the work being done by the consumer, whether that be a new `Observable` or our final `Subscriber`. The orange code shows the work being scheduled by the `subscribeOn` operator and the blue code shows the work being scheduled by the `observeOn` operator.

We may actually want to apply multiple `observeOn` operators in our chain. Here we start doing work on the `Executor`, followed by some work on another background thread provided by `computation`, followed lastly by the main thread.

While this is fine to do with `observeOn`, it shouldn't be done with `subscribeOn`. This is because `subscribeOn` only effects how the source `Observable` is scheduled and any other `subscribeOn` operators below it will get overshadowed and therefore have no effect. This means we can move the `subscribeOn` around in our chain and it has no effect on which work gets scheduled where.

`AndroidSchedulers.mainThread` is actually not part of RxJava itself. You have to pull in a separate dependency called [RxAndroid](https://github.com/ReactiveX/RxAndroid). As an Android developer it's common having to switch back and forth from the main thread, so you will most definitely want to include it.

[Slide 55-56](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=55):

Some `Observables` are actually implicitly async, so be careful of this. For example, the [`interval`](http://reactivex.io/documentation/operators/interval.html) operator returns a new `Observable` that emits an incrementing `Long` value (starting at zero) each time the specified duration elapses. In order to do this, RxJava automatically schedules this work for us on a background thread. We can optionally specify which `Scheduler` to run on by passing in a third argument (it uses `computation` by default).

[Slide 57-58](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=57):

I know everyone tests their code (right?) and probably want to know how we test an asynchronous stream.

Here's a method we want to unit test. It returns an `Observable` that will emit some Android versions to us and it does this work on a new thread.

[Slide 59](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=59):

One way we might approach this is by doing the following:

We create a list of versions for storing the values generated by the `Observable`. Then when we call the `getSupportedVersions` method, we immediately apply the `toBlocking` operator to the returned `Observable`. This returns us a new type of `Observable` called a [`BlockingObservable`](http://reactivex.io/RxJava/javadoc/rx/observables/BlockingObservable.html), which waits for the `Observable` to complete before proceeding on the current thread. Then we can subscribe to the `BlockingObservable`, store each value emitted to our list and then perform any necessary assertions on it.

[Slide 60](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=60):

RxJava has built-in support for testing, so there's actually a better way of doing this. We can use a [`TestSubscriber`](http://reactivex.io/RxJava/javadoc/rx/observers/TestSubscriber.html) and pass that to the `subscribe` method. This will automatically block the `Observable` it's subscribed to. Moreover, it provides built in assertion methods specific to the `Observable` type. We can assert which values were emitted, if there were any errors, if it's completed, etc...

[Slide 61-65](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=61):

Another built-in type for testing is [`TestScheduler`](http://reactivex.io/RxJava/javadoc/rx/schedulers/TestScheduler.html). I mentioned before that one of the things a `Scheduler` does is determine when something happens. The `TestScheduler` will allow us to control time and therefore trigger when certain events happen.

This technique is useful for certain types of `Observables`, such as those on a repeated timer like `interval`. We can either pass in our `TestScheduler` as the extra argument to `interval`, or apply it with the `observeOn` operator to the resulting `Observable`.

We can then control the timing of events in the `Observable` under test. Initially we expect no values to have been emitted. But after time has advanced by 2 seconds, we expect to see the value `0L`.

[Slide 66-68](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=66):

So for this talk I went ahead and wrote [a little sample app](https://github.com/jpetitto/rxjava-android-example) that uses RxJava.

You can clone the project from my GitHub:

{% highlight bash %}
git clone https://github.com/jpetitto/rxjava-android-example.git
{% endhighlight %}

The app allows you to search for users on GitHub. It makes a call to the GitHub API and then displays the results in a simple `RecyclerView` When you click on a user listed in the search results, it triggers a `Toast` to appear with their username.

[Slide 69-74](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=69):

The code here is the core of the app`s functionality. It takes the user's search query and displays the results on the screen.

First thing we do is create an `Observable` from our search bar, which is just an `EditText`. We're using [RxBinding](https://github.com/JakeWharton/RxBinding) here, which allows us to consume most (all?) of the Android widgets in a reactive manner. In this case, we're creating an `Observable` that will fire an event each time the text in our search bar changes.

Once we have the text from the resulting text change, we want to immediately move off of the main thread  to do all of the necessary work related to the search. I've chosen `Schedulers.io` here since we'll be performing a network call to the GitHub API.

The [`debounce`](http://reactivex.io/documentation/operators/debounce.html) operator will only emit the most recent notification once there's been no new notifications for the specified duration. We can use this so that we don't perform a new search query every time the user types in a new letter; we'll only perform it once they've paused from entering input for 1 second.

We'll go ahead and apply a `filter` that will discard any empty search queries. For instance, `RxTextView.textChanges` will emit an event for the search bar's initial state, which will be empty.

Now we'll actually perform the network call to obtain a response for our search query. I made a class called `GitHubInteractor` that's responsible for interacting with the GitHub API.

[Slide 75-85](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=75):

We're using [Retrofit](http://square.github.io/retrofit/) to help us with the network call. If you haven't used Retrofit before, it's essentially a wrapper around an HTTP client, providing automatic conversion of requests and responses to our data model objects. With little more than an interface declaration, we can make a call to to the GitHub API with our search query and get back a `SearchResult`. What's even cooler is that Retrofit can actually return an `Observable` instead of having to use a callback.

`searchUsers` is the method on `GitHubInteractor` that we call into. We'll use the `concat` operator to first check the cache in case we previously performed the same search query. Otherwise we'll make the network call to fetch the results. The [`first`](http://reactivex.io/documentation/operators/first.html) operator will prevent the network `Observable` from being subscribed to, and thus performing a network call, when the cache `Observable` emits a result.

The `cachedObservable` method simply creates a new `Observable` from the value in the cache. In the case that the cache returns null, meaning there's no cached result, we'll filter it out. This will prevent a value from being emitted by the cached `Observable` before it completes, therefore making `concat` subscribe to the network `Observable`.

`networkObservable` calls the `searchUsers` method on our `GitHubService` interface, returning the `Observable` that Retrofit creates for us on our behalf. We also need to store this result in the cache in case the same query is used again. We can use the `doOnNext` operator to achieve this, which is intended for side effects. There are [similar operators](http://reactivex.io/documentation/operators/do.html) for `onError` and `onCompleted`, for example.

We build our `Retrofit` object, configuring it to point to the GitHub API. Then we inject it into `GitHubInteractor` along with the cache. Note that in order for Retrofit to convert the response into an `Observable` we have to specify the call adapter factory. This is a separate dependency in Retrofit 2 and must be specified in your app's `build.gradle` file.

[Slide 86-91](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=86):

A nice thing about obtaining the `Observable` through the interactor is that we don't care how its values are being produced. We don't necessarily care if the data is coming from the cache or the network, just that we get an `Observable` back that generates a `SearchResult`. This decouples the producer from the consumer, leading to increased separation in our code.

We're using [`switchMap`](http://reactivex.io/RxJava/javadoc/rx/Observable.html#switchMap(rx.functions.Func1)) here, which behaves similar to `flatMap`, but will unsubscribe from the `Observable` generated for the previous emitted value. We don't want an old search result coming in if we're going to make another query.

Next we want to sanitize our search results before displaying them to the user. First we create a new `Observable` from the `SearchItems` embedded inside the `SearchResult`. An individual `SearchItem` simply represents a GitHub user in this case. Then we use the `limit` operator to prevent the user from seeing more than 20 items at a time. Finally we convert this into a `List` so our `Subscriber` can consume the search results as a single value.

Now that we have our desired results, we are ready to move back to the main thread. Once we invoke `subscribe`, the work will start on the main thread since we're calling it from the `MainActivity`, which is running on the main thread. We take each `List<SearchItem>` that gets emitted and update the adapter of our `RecyclerView` with it. As far as errors are concerned, we're just going to log them in the example.

[Slide 92-94](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=92):

You probably noticed that we're assigning the result from `subscribe` into this `subscription` variable. Having reference to a [`Subscription`](http://reactivex.io/RxJava/javadoc/rx/Subscription.html) allows us to clean up any resources associated with a non-completed `Observable` when we longer need to consume it.

In the example we're creating an `Observable` with the `RxTextView.onTextChanges` method, which takes a reference to our `EditText` that we pass in. Since the `Observable` has a strong reference to the `View` and the `View` has reference to its `Activity`, we would actually end up leaking the `Activity` if we don't `unsubscribe`.

So it's common in Android to `unsubscribe` from an `Observable` in the `onDestroy` method of an `Activity`. This will prevent us from leaking memory when our `Activity` gets destroyed. There is also a [`CompositeSubscription`](http://reactivex.io/RxJava/javadoc/rx/subscriptions/CompositeSubscription.html) which allows us to store multiple `Subscription` objects and then unsubscribe from each simultaneously.

Note: The example project also contains a reactive event bus for displaying the username as a `Toast` when a user is clicked on in the adapter. There are also unit tests to show how we might test this code.

[Slide 95](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=95):

So there's a few big things in the pipeline for RxJava. The first is the [`Single`](http://reactivex.io/documentation/single.html) type, which can only ever emit a single value. This is common when making network calls where there is a single request and response. Instead of having to deal with multiple values and `onCompleted`, we have a simpler set of semantics for dealing with asynchronous calls. The nice thing about `Single` is that it's composable with the `Observable` type and vice versa. It's currently marked as beta but has been worked on for quite some now.

[RxJava 2.0](https://github.com/ReactiveX/RxJava/tree/2.x#version-2x), which is also in beta, is ramping up for its official release. It takes advantage of the latest features from Java 8 in order to help simplify its implementation. It follows the [Reactive Streams](http://www.reactive-streams.org/) specification, which describes a standard for asynchronous programming with streams, so there's some fairly significant API changes as well. As Android developers we don't need to worry about the 1.x version becoming dormant, as it's still going to be supported in parallel with the 2.x version.

[Slide 96](https://speakerdeck.com/jpetitto/rxjava-for-android-developers-austin-droids-meetup?slide=96):

Here's a collection of resources you may find useful. I recommend checking out the last one, [RxMarbles](http://rxmarbles.com/), as it provides a cool mechanism for visualizing and interacting with the different operators in RxJava. This can help your understanding of how certain operators in the library behave.
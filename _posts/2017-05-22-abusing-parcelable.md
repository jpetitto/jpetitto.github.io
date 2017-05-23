---
layout: post
title: Abusing Parcelable
comments: true
permalink: abusing-parcelable
---

<!-- excerpt.start -->
[I posted an article](parcelable) a few months back explaining some of the issues of using Parcelable and how using a library can help mitigate them. I've since come to the realization that these issues should have never really arisen in the first place. The real issue was that I was using Parcelable the wrong way.

It might seem easy, and even appropriate, to generate a Parcelable implementation for most of your data objects. The ones that get used by your views may need to get passed to an Intent or Bundle in order to be available to another Activity or Fragment.
<!-- excerpt.end -->

But this is the wrong way of looking at things. You shouldn't be passing around entire static objects like this between components. Instead, you should just be passing around the minimum amount of data required (e.g. an ID) to lookup the actual object. The assumption is that you would be fetching this object from a local data cache, whether that be in memory or via a database.

This has quite a few benefits for your application. For one, the data is no longer static and isn't prone to becoming stale. Your view can fetch the object each time it gets recreated. If you're using an event bus, an `Observable`, or the newly minted [LiveData class](https://developer.android.com/topic/libraries/architecture/livedata.html) from the Android framework, you can freely receive updates for that specific object based on its meta data. You could do this in tandem with a static object, but what's the point of that if you are already writing the code to receive live updates for it.

Another benefit is that some frameworks and libraries don't work well with objects that implement Parcelable. Realm is one of these frameworks. While you could implement both `RealmModel` and `Parcelable` on your class, it really doesn't make a whole lot of sense. Objects managed by Realm shouldn't be parceled and creating a copy of the object defeats many of the benefits of Realm.

The last benefit is that it's possible to parcel too much data at once. If you have a lot of data being passed between your views, then the Android `Binder` class will [throw an exception](https://developer.android.com/reference/android/os/TransactionTooLargeException.html) when it exceeds 1MB of data. We actually had crashes in our apps due to this and it was not something we were able to catch until our users got their hands on our apps.

Parcelable can be useful for passing around small amounts of static data, but ultimately you should only be passing around the key information for fetching the actual data. You'll soon find that you'll rarely need to work with Parcelable at all.

<sup>If you don't currently persist data locally, check out [Realm](https://realm.io/docs/java/latest/) or Google's new [Room library](https://developer.android.com/topic/libraries/architecture/room.html) (there's always plain old SQLite too).</sup>
---
layout: post
title: Use a Library when Parceling Data
comments: true
permalink: parcelable
---

<!-- excerpt.start -->
As an Android developer you've probably used Parcelable for one thing or another: whether it's storing an object in an Intent or passing arguments to a Fragment. While there are ways to avoid using Parcelable, it can often be the simplest and quickest way of passing data around.

Implementing Parcelable is pretty straightforward. In fact, Android Studio can auto generate the code for you when you add the interface to your class.
<!-- excerpt.end -->

<div style="text-align: center;"><img src="../assets/parcelable.gif"></div>

There are third party libraries which leverage annotation processing (check out [Parceler](https://github.com/johncarl81/parceler) and [AutoParcel](https://github.com/frankiesardo/auto-parcel)<sup>1</sup>) to generate this code for you at compile time. Why might you want to use one of these when Android Studio can simply generate the code for you?

The first reason is a bit superficial but still important: it reduces the noise in the class's implementation. You need to have a static `Creator` member, a specific constructor for reading the Parceled data, a method for writing the Parceled data, and an often unused `describeContents` method. None of this is important to what the class does; we only care that it is Parcelable.

The second and more important reason is modification. What happens if we add or remove a field from our class? We have to remember to update our Parcelable logic. We also have to ensure the ordering of the fields being read in matches the order that we write to. A slip-up here can cause exceptions or unexpected behavior. With an annotation processor, when we change our class, the generated Parcelable code will change with it.

Yes, generating the Parcelable code is about as fast with Android Studio as it is with an annotation library. The difference with using a library though is that it makes your code both cleaner and less prone to errors.

<sup>1. AutoParcel has the added benefit of working with [IcePick](https://github.com/frankiesardo/icepick) for saving instance state.</sup>
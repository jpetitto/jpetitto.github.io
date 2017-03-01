---
layout: post
title: Use a Library for Parceling Data
comments: true
permalink: parcelable
---

<!-- excerpt.start -->
As an Android developer you've probably used Parcelable for one thing or another: whether it's storing an object in an Intent or passing arguments to a Fragment. While there are ways to avoid using Parcelable, it can often be the simplest and quickest way of marshalling data across process boundaries.

Making a class implement Parcelable is pretty straightforward. In fact, Android Studio will auto generate the code for you when you add the interface to your class's declaration.
<!-- excerpt.end -->

<div style="text-align: center;"><img src="../assets/parcelable.gif"></div>

There are third party libraries which leverage annotation processing (check out [Parceler](https://github.com/johncarl81/parceler) and [AutoParcel](https://github.com/frankiesardo/auto-parcel)*) to generate this code for you at compile time. Why might you want to use one of these when Android Studio can simply generate the code for you?

The first reason is a bit superficial but still important: it reduces the noise in the class's implementation. You need to have a static Creator member, a specific constructor for reading the Parceled data, a method for writing the Parceled data, and an often unused describeContents method. None of this is important to what the class does; we only care that it is Parcelable.

The second or more important reason is that this approach is error prone. What happens if we add or remove a field from our class? We have to remember to update our Parcelable logic. We also have to ensure the ordering of the fields being read matches the ordering that we write to. A slip-up here can cause exceptions or unexpected behavior.

Yes, generating the Parcelable code is about as fast with Android Studio as it is using an annotation library. The key benefits of using a library though is that it makes both your code cleaner and less error prone to errors.

* AutoParcel has the added benefit of working with IcePick for saving instance state.
---
layout: post
title: Fragments, Don't Hurt Me
comments: true
permalink: fragments-hurt
---

<!-- excerpt.start -->
Here’s a screen I was recently working on for an app at work:

<div style="text-align: center;"><img src="../assets/viewpager-good.png"></div>

It’s built with a `Fragment` containing a `ViewPager`. Each page in the `ViewPager` is served by a `FragmentPagerAdapter`. Pretty straight forward stuff.

On first launch it works as expected, but if the enclosing `Activity` is destroyed, no pages get recreated by the `FragmentPagerAdapter`.<!-- excerpt.end -->

<div style="text-align: center;"><img src="../assets/viewpager-bad.png"></div>

Here’s the code for creating the `ViewPager` (shortened for brevity):

{% highlight java %}
public class TPGTrackerFrag extends Fragment {
  @Override
  public View onCreateView(...) {
    ViewPager viewPager = (ViewPager) v.findViewById(R.id.tracker_view_pager);
    PagerAdapter adapter = new TrackerPagerAdapter(getFragmentManager());
    viewPager.setAdapter(adapter);

    TabLayout tabLayout = (TabLayout) rootView.findViewById(R.id.tracker_tabs);
    tabLayout.setupWithViewPager(viewPager);
  }
}
{% endhighlight %}

And the code for the `TrackerPagerAdapter` (also shortened for brevity):

{% highlight java %}
private class TrackerPagerAdapter extends FragmentPagerAdapter {
  public TrackerPagerAdapter(FragmentManager fm) {
    super(fm);
  }

  @Override
  public Fragment getItem(int position) {
    // returns a Fragment for each tab
  }
}
{% endhighlight %}

Can you see what’s wrong?

It turns out the issue lies with the `FragmentManager` passed to the `FragmentPagerAdapter`. If we instead pass in the `Fragment`'s child `FragmentManager`, our code works as expected.

{% highlight java%}
PagerAdapter adapter = new TrackerPagerAdapter(getChildFragmentManager());
viewPager.setAdapter(adapter);
{% endhighlight %}

A subtle but important difference. The `FragmentManager` returned by `getFragmentManager()` is bound to the enclosing `Activity`, while `getChildFragmentManager()` is tied to the `Fragment` itself. Since the `Fragment` is loaded into the `ViewPager` are nested within our `Fragment`, we must use the child `FragmentManager` or we’ll see weird behavior like this.

We can forgo `Fragment`s altogether by using a `PagerAdapter` to load views. This is how `ViewPager` was typically used before `Fragment`s were introduced into the framework. If you’re looking to move away from `Fragment`s entirely, I suggest checking out [Conductor](https://github.com/bluelinelabs/Conductor).

In our app we’ve hit similar snags with `Fragment`s too. Issues around adding the same `Fragment` instance to a container, unexpected lifecycle events with the back stack when using `add` vs. `replace`, and a handful more have arisen (dive into [this r/AndroidDev thread](https://www.reddit.com/r/androiddev/comments/53ca54/fragments_what_are_they_good_for/) to read about many of them).

Generally speaking, it’s not very hard to get in trouble with the `Fragment` APIs. Their behavior is often unexpected and this tends to compound as the complexity of your app grows.
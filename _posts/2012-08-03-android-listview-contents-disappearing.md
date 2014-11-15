---
title: Android ListView Contents Disappearing
author: error454
layout: post
permalink: /2012/08/03/android-listview-contents-disappearing/
tagazine-media:
  - 'a:7:{s:7:"primary";s:55:"http://mobilecoder.files.wordpress.com/2012/08/gone.jpg";s:6:"images";a:2:{s:55:"http://mobilecoder.files.wordpress.com/2012/08/gone.jpg";a:6:{s:8:"file_url";s:55:"http://mobilecoder.files.wordpress.com/2012/08/gone.jpg";s:5:"width";i:540;s:6:"height";i:774;s:4:"type";s:5:"image";s:4:"area";i:417960;s:9:"file_path";b:0;}s:58:"http://mobilecoder.files.wordpress.com/2012/08/working.jpg";a:6:{s:8:"file_url";s:58:"http://mobilecoder.files.wordpress.com/2012/08/working.jpg";s:5:"width";i:540;s:6:"height";i:779;s:4:"type";s:5:"image";s:4:"area";i:420660;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:2;s:6:"author";s:8:"11758919";s:7:"blog_id";s:8:"11929434";s:9:"mod_stamp";s:19:"2012-08-03 17:52:05";}'
categories:
  - Android
  - Interesting Problems At Work
tags:
  - android
  - arabic
  - disappear
  - listview
---
<img src='{{ site.url }}/assets/uploads/2012/08/gone.jpg' alt=''>

Sometimes you encounter issues on Android that seem so blatantly simple, you can't imagine that you are the first person to have hit them.  This seemed like a simple case where my TextView items were not being rendered sometimes when I scrolled my ListView.
<!--more-->


## Stackoverflow to the Rescue

In typical programmer fashion, this is the first place I turned where I found that I was in good company.  Unfortunately all of the people encountering the issue on SO had been hiding elements in their List and were forgetting to unhide them.  I wasn't doing any tricks in my CursorAdapter, just setting the raw data to the TextView.  There was something out of the ordinary for my case, in that some of my TextViews were using arabic characters, some were using western characters while others were mixing arabic and western.

## Hierarchy Viewer to the Rescue

In typical Android fashion, I then turned to the tool we all use when something is amiss with our layouts.  Hierarchy Viewer!  One of many tucked away tools in the SDK.  Frustratingly, if an Arabic TextView was present in my layout, Hierarchy Viewer would not run!  This led me to file a <a href="http://code.google.com/p/android/issues/detail?id=35890" target="_blank">bug against the hierarchy viewer</a>.

Ok, no problem, I'll just inspect a blanked out view that doesn't have any arabic in it.  Now when I run hierarchy viewer, the act of running hierarchy viewer fixes my view?!?

Pop quiz, what are 2 things that hierarchy viewer does to every element in your layout?

1.  Calls **invalidate**
2.  Calls **requestLayout**

A simple change to my CursorAdapter to requestLayout after setting the text solved the problem.

[<img class="alignnone size-full wp-image-1142" title="working" src="{{ site.url }}/assets/uploads/2012/08/working.jpg" alt="" width="540" height="779" />][1]

## Note to Arabic Developers

From the arabic proverb

> A horse that will not carry a saddle must have no oats.

Or perhaps the more familiar version

> ويجب على الحصان الذي لن يحمل سرج ليس لديهم الشوفان.

What does this have to do with Android?  Absolutely nothing!  But if you plan on using both of these in a ListView, prepare to **requestLayout **or there will be much tearing of clothes and gnashing of teeth.

 [1]: {{ site.url }}/assets/uploads/2012/08/working.jpg
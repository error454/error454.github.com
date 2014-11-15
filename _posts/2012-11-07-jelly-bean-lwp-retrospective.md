---
title: Jelly Bean LWP Retrospective
author: error454
layout: post
permalink: /2012/11/07/jelly-bean-lwp-retrospective/
publicize_twitter_user:
  - error454
publicize_reach:
  - 'a:2:{s:7:"twitter";a:1:{i:304924;i:132;}s:2:"wp";a:1:{i:0;i:45;}}'
categories:
  - Android
  - Java
  - 'Marketplace  Publishing'
  - Shiva 3D
tags:
  - android
  - play market
  - retrospective
  - shiva
  - shiva3d
  - wallpaper
---
My brother and I have been working on a game for the upcoming OUYA console.  We got a little burnt out on a particular design issue and needed a breather, a small fun project to lift our spirits.  So we decided to try our hand at a Live Wallpaper for Android.  1 week later and here we are.
<!--more-->


Coming into this project, I knew it would be challenging, there were many things I had never touched before.  As always with programming, much of your success comes from standing on the shoulders of others.  This project was an exemplary example of that.

The actual 3D rendering, physics system and lighting are provided by the ShiVa 3D engine.  It took about 2 days to write the ShiVa game that handles switching camera views, adding forces to jelly beans and triggering animations.  My brother did all the modeling and animation.

## Live Wallpaper Integration

To integrate ShiVa into an Android app, you typically just export the project which generates a bunch of Java/JNI that you don't really need to care about.  I have an open source project that integrates various SDKs into this exported code so I was already familiar with the implementation details.  ShiVa basically provides a GLSurfaceView implementation, like any openGL engine would.  Unfortunately, Live Wallpapers don't use GLSurfaceView, they instead integrate a WallpaperService Engine which does nothing more than hand you a surface to draw on.  I needed a way to convert the GLSurfaceView and Renderer over to the live wallpaper engine and I did not feel like writing a GL thread from scratch.  The main reason I use ShiVa is so that I can spend my time designing my game, not my engine.

[GLWallpaperService][1] to the rescue!  This open source project helped immensely, it took around 3 days of work to perfect the whole ShiVa/Wallpaper integration.  The list of tasks were:

*   Slightly modifying the ShiVa GLSurfaceView.Renderer to use GLWallpaperService
*   Moving the ShiVa engine callbacks into a Wallpaper Engine
*   JNI to handle screen changing events
*   JNI to handle engine initialization and wallpaper configuration
*   Solving concurrency issues with the ShiVa engine (Live Wallpapers can run two instances of your wallpaper simultaneously, something that the engine was having an issue with).

On that last bullet point, sometimes limitations have to be worked around in a creative way.  The problem is that the ShiVa engine can’t run two instances at a time because of some file locking that occurs inside the engine (to the best of my debug ability).  But, a live wallpaper can be running in the background AND be displayed in a preview window.

To overcome this, I made the decision of rendering a static image/text on the preview window.  It’s kind of cheap but it gets the job done.

<a href='{{ site.url }}/assets/uploads/2012/11/loveme.png'><img src='{{ site.url }}/assets/uploads/2012/11/loveme.png?w=300' alt=''></a>

Looking back over my commits for the last 7 days, I can easily find the one that marks the completion of the ShiVa integration as a live wallpaper:

[<img class="alignnone size-full wp-image-1187" title="github" alt="" src="{{ site.url }}/assets/uploads/2012/11/github.png" height="77" width="603" />][2]

## Android Experience

The next step was the Android experience.  As I looked around at live wallpapers, I noticed that very very few live wallpapers:

*   Installed an app in the launcher
*   Had a settings page
*   Made an effort to have a nice android UX

I wanted a classy ICS experience with a settings screen front and center, ability to hit my social network pages and some smooth sexy scrolling.

Obviously the first thing I did was use [ActionBarSherlock][3].  Really, this is a no-brainer and I’m surprised that Google hasn’t replaced the support package with ABS by now.  I made a couple visual layout widgets, scribbled out a few designs and went to work.  This was one of the most time consuming tasks just because I iterated on the design several times.

The first issue was that I wanted the sexy ICS on/off switches.  You can see below that I got them, backwards compatible down to android 2.1 thanks to an awesome open source library called [android switch widget backport][4].

<a href='{{ site.url }}/assets/uploads/2012/11/screenshot_2012-11-07-01-31-27.png'><img src='{{ site.url }}/assets/uploads/2012/11/screenshot_2012-11-07-01-31-27.png?w=300' alt=''></a>

The next thing I wanted was a nice color picker.  Once again a few minutes on google uncovered a gem called [devmil android color picker][5].  I made a few hacks and was quite pleased with the results.

<a href='{{ site.url }}/assets/uploads/2012/11/screenshot_2012-11-07-01-31-41.png'><img src='{{ site.url }}/assets/uploads/2012/11/screenshot_2012-11-07-01-31-41.png?w=300' alt=''></a>

That’s a total of four amazing Apache license based projects that cut tons of time off of this 1 week project.  My design skills are not amazing but this is definitely the best looking live wallpaper settings app that I have ever seen AND it looks great on the handset as well as the tablet.

[<img class="aligncenter size-medium wp-image-1183" title="Screenshot_2012-11-07-01-32-00" alt="" src="{{ site.url }}/assets/uploads/2012/11/screenshot_2012-11-07-01-32-00.png?w=300" height="225" width="300" />][6]

## Marketing

After wrapping up the Android interface and wiring everything together, I had to do the stuff I hate, marketing.  One of my deficits that I’m continually trying to work on.  This is the step of creation that I always forget about and that I struggle to get through.  But yet it is kind of essential if you want to make any return on your work.

So I sat down and did my best to write some good app store copy.  I struggled to get good screenshots to use in the feature graphic.  I sat in photoshop for a few hours, bothering my brother at one point to make some cool looking text.   I ended up with this.

<a href='{{ site.url }}/assets/uploads/2012/11/feature-graphic.png'><img src='{{ site.url }}/assets/uploads/2012/11/feature-graphic.png?w=300' alt=''></a>

The goal was simple, make a graphic that children can’t NOT click on.  Wife test passed, 3-year old niece passed, I’m too exhausted to add shadows, let’s ship this stupid thing!

## Final Thoughts

It always amazes me how complicated even simple projects can be.  This was a difficult project to get right that spanned Java, C++ and LUA.  I didn't mention the 1.5 days spent writing a licensing and in-app purchase verification server prototype in Google App Engine (that I later abandoned) or the 0.5 days spent mastering my native build process and proguard  some details are even too boring for me to talk about.  Now that I've taken care of the boring framework, I can spend time writing/updating live wallpapers!

 [1]: https://github.com/markfguerra/GLWallpaperService
 [2]: {{ site.url }}/assets/uploads/2012/11/github.png
 [3]: http://actionbarsherlock.com/
 [4]: https://github.com/BoD/android-switch-backport
 [5]: http://code.google.com/p/devmil-android-color-picker/
 [6]: {{ site.url }}/assets/uploads/2012/11/screenshot_2012-11-07-01-32-00.png
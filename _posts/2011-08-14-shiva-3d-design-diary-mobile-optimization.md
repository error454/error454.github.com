---
title: 'Design Diary  Shiva 3D Mobile Optimization'
author: error454
layout: post
permalink: /2011/08/14/shiva-3d-design-diary-mobile-optimization/
tagazine-media:
  - 'a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";s:1:"0";s:6:"author";s:8:"11758919";s:7:"blog_id";s:8:"11929434";s:9:"mod_stamp";s:19:"2011-08-13 21:40:45";}'
categories:
  - Android
  - Shiva 3D
  - WebOS
tags:
  - audio
  - design
  - openal
  - performance
  - pools
  - sensors
  - shiva
  - shiva3d
  - slowdown
  - texture
  - textures
---
As we are narrowing in on the release of our game, we turned our attention on the game performance.  Our low end target is an Android HTC Inspire, mid-range is an HP Pre Plus and super high-range is an HP Touchpad.

We were having an issue with 1 scene in-particular where on the Touchpad, we were consistently dropping to 15 fps. For a dual core beast, this was bad.

This article is partly about the debug process and highlights 3 areas we found where we gained performance.

<h1 class="more">
  Finding the problem area
</h1>

It was difficult to pinpoint the problem area.  The question I was trying to answer was whether we were cpu or memory starved.  I began my investigation by opening 2 consoles on the Touchpad, running top in one and executing the game in the other.  I discovered that we were pegging the CPU.  So I began to wonder:

*   Do I have a runaway AI that is churning cpu?  Perhaps I'm forgetting to set something into the idle state.
*   Do I have a log statement in one of my sensor collision handlers that is causing an I/O bottleneck?
*   Do one of my hidden objects still have their dynamics engine enabled?
*   Is my AI navigation algorithm too costly?
*   Are the quantity of objects using dynamics simply too much to handle?
*   Is my dynamics iteration count too large?

So I began tearing things apart 1 at a time:

1.  Disable all AI in the scene
2.  Disable all dynamics in the scene
3.  Disable all sensors in the scene

At this point I had a static scene with 3 objects in it and the performance problem persisted?!

Then I started setting objects to be invisible in the scene, 1 at a time.  Here I discovered that a single 300 vertice object was the source of this scene's problem.

# Textures

The problem object had 3 textures assigned to it.  Each texture was 10241024. To make matters worse, these textures were completely flat colors that could be replaced by setting the diffuse color in the material lighting.

By not using ridiculous texture sizes for things that werent even textures, we solved a major performance problem.  After seeing this, I browsed through our entire texture library and deleted any textures that were just a flat solid color.  I knew this would cause shiva to crash when a material using the deleted texture was used.  Anticipating this, I grep'd through the materials folder, found all materials using these textures and fixed them (removing the texture and changing diffuse lighting).

# Audio Performance

We noticed that on our low end target, every time we had music playing or fired the gun, the framerate would drop by 10 fps.  Not only that but the max fps topped out around 18  horrible!  What we discovered is that audio can create a huge performance bottleneck on android.  There are 2 aspects to audio on android, the audio subsystem and the audio you are puting through it.

First, we were using 44 kHz stereo mp3s for music and 44 kHz mono mp3s for sound effects.  In shiva, when you do **sound.play**, the whole audio file is thrown into memory.  You can imagine that if the file is big and memory is low, this could be a problem.  Alternatively, using **music.play** will stream the music rather than load the entire file into memory.

Our first problem was the music, given the sample rate, this was simply pushing more bits into memory than the phone could handle.  We resampled the music down to mono 22 kHz and overall performance in the scene improved significantly!  However, this did not solve the slowdown we experienced when firing the gun (sound.play).

After much debugging, we discovered that when you export a game in Shiva, you are given the choice of which audio sub-system to use, OpenAL or Android.  At this point, all I know is that OpenAL produces HORRIBLE performance on the HTC Inspire (Android 2.2)

# Sensor Performance

After more trial and error testing, I discovered that having approximately 15 overlapping sensors firing was pegging the CPU.  This surprised me because for testing, I had short circuited the sensor handlers so that they returned immediately.  So just the overhead of the sensors being called was destroying performance.  This was due to using onSensorCollision where we really didn't need to be, see my <a href="http://mobilecoder.wordpress.com/2011/08/11/design-diary-shiva-3d-sensor-performance/" target="_blank">sensor post</a> for more details.

# Object creation and deletion

The final area that we found issues was dynamic runtime object creation and deletion.  We would notice a slight pause every time and enemy was destroyed because for every enemy death, we need to create an explosion object.  On the low end target , we also noticed that spawning a group of X drones would cause the same blips after around the 5th drone.

I thought that just by forcing models to stay loaded that I would work around these issues, but this didn't seem to be the case.  I solved this problem by creating pools of objects and some simple object allocation/deallocation handlers that initialize and return objects to the pool.  I like this architecture because now I can set the limit to the number of decals or ships in a scene.  With object pools, you take the hit at the beginning of the scene instead of at the time of creation.
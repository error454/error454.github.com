---
title: Profiling OUYA (Tegra 3) Games using Nvidia PerfHUD ES
author: error454
layout: post
permalink: /2013/11/14/profiling-ouya-tegra-3-games-using-nvidia-perfhud-es/
categories:
  - Android
  - Graphics
  - OUYA
  - Shiva 3D
  - Tegra 3
tags:
  - directed tests
  - ouya
  - perfhud
  - shiva
  - tegra
---
PerfHUD ES has a good <a href="http://docs.nvidia.com/tegra/index.html#PerfHUD_ES_User_Guide.html" target="_blank">online manual</a> that I'm not trying to replicate.  The manual explains the UI, graphs and knobs. The purpose of this article is to introduce the tool to people that aren't aware of it as well as to provide more details behind the *directed tests* section.  Specifically, how to make your game faster based on the results you get from the directed tests.

<img src='{{ site.url }}/assets/uploads/2013/11/perfhud.png' alt='Developing on Tegra and not using perfHUD ES is like eating a french dip sandwich with no sandwich.'>

To enable perfhud, in a shell run:

<pre>adb shell setprop debug.perfhudes 1</pre>

Then fire up your game, launch perfhud and connect to your device.

## The Frame Debugger

The frame debugger is so incredibly useful and awesome that it's hard to describe it all.  Here are the top reasons.

### Seeing every draw call

When you see every single draw call and exactly what it is drawing it is amazing.  You will discover things that you didn't know were in your scene, things that shouldn't be visible and other amazing stuff.

<a href='{{ site.url }}/assets/uploads/2013/11/frame-scrub1.png'><img src='{{ site.url }}/assets/uploads/2013/11/frame-scrub1-1024x763.png' alt='Seeing every single draw call flash on your screen is.... awesome!'></a>

### Inspecting Geometry

Yes, you can fully inspect geometry.  This can be incredibly useful for validating your mesh frustum culling.  You can even take a peak at games that you don't have source to and reverse engineer special FX!

<a href='{{ site.url }}/assets/uploads/2013/11/model-view.png'><img src='{{ site.url }}/assets/uploads/2013/11/model-view-1024x759.png' alt='Inspecting a model.'></a>

### Validating Mipmaps

With the texture viewer, you can validate mipmaps and check your texture compression.  Seriously, amazing.

<a href="{{ site.url }}/assets/uploads/2013/11/texture-view.png"><img src="{{ site.url }}/assets/uploads/2013/11/texture-view-1024x758.png"></a>

### Hot-Replacing Shaders

You can view all the shaders used in your game in this tab.  You can see exactly which draw call(s) use them, how expensive the shaders are and can even replace the shaders on the fly.

I found this incredibly useful when tracking down expensive shaders.  I would simply hot-replace a shader with a barebones shader to see if my performance increased.

<a href="{{ site.url }}/assets/uploads/2013/11/shaders.png"><img src="{{ site.url }}/assets/uploads/2013/11/shaders-1024x761.png"></a>

## Directed Tests

The directed tests area is where you'll solve most of your performance problems.  The test methodology here is:

<a href='{{ site.url }}/assets/uploads/2013/11/directed-tests-cross.jpg'><img src='{{ site.url }}/assets/uploads/2013/11/directed-tests-cross-175x300.jpg' alt='We won&apos;t be covering the crossed out items.  If you need them, you already know what they do.'></a>

1.  Look at FPS (at the bottom of the screen) 
    1.  FPS is displayed as Current FPS/10 second Avg FPS
2.  Click a directed test
3.  Look at FPS

If your FPS made a noticeable increase, then you now know what area your problem is in.  If not, continue to the next test.  We're not going to cover all of these because frankly as someone who uses a game engine, I've never found the last 5 options helpful in solving performance issues. For each of these checkboxes, we are going to answer 2 fundamental questions:

1.  **What does the checkbox do?**
2.  **If the checkbox increased framerate, what should we do?**

Note that performance is a very complicated subject.  Anytime you see the word is replace it with is most likely  and imagine me making little quotation signs in the air with my hands.

## Textures

### What it does

Replaces all game textures with a texture measuring 2 pixels x 2 pixels.

### What to do

If this increases your framerate then your bottleneck is most likely in the texture unit. Look at eliminating this bottleneck by the most common methods:

*   Reduce texture count by combining multiple textures into a single texture atlas
*   Reduce texture size e.g. from a max of 20482048 to 10241024
*   Use mip-maps, otherwise your 11 pixel object still uses a 10241024 texture!
*   Use a faster texture filtering method, e.g. bilinear instead of anisotropic filtering
*   Increase texture compression

## Ignore Draw Calls

### What it does

Stops drawing to the screen by ignoring all DrawArrays and DrawElements calls.  Essentially simulates having the fastest possible GPU that could ever exist.  This is a good test to determine whether you might be CPU bottlenecked.

### What to do

There are certainly 2 sides to the draw call coin.  If FPS does not improve when checking this, you have a CPU bottleneck.  If you are CPU bound then you need to take a good look at your code.  Basically stop here, you need a different set of profiling tools to help you find the code at fault.  Look to your debugger or do a simple pause test to track down the offending code and then apply optimization recommendations for your specific language.

If FPS improves then you have a GPU limitation.  Some of the other tests will hopefully narrow down which specific GPU bottleneck you have.  If you've exhausted the other GPU tests without any improvement, then you may fall into 2 additional cases.

#### **Poly Count**

You are simply drawing too much geometry.  You should try to:

*   Reduce your mesh and/or UV complexity
*   Use frustum culling for large meshes
*   Use Level Of Detail to reduce mesh complexity at a distance

#### **Draw Calls**

You are trying to draw too much in a single frame.  You should:

*   Combine multiple meshes into a single mesh (possibly at runtime)
*   Use batching (use the batching histogram to help determine if you aren't batching efficiently)
*   Remove frivolous objects for lower-end devices (ocean simulations and animated foliage come to mind)

## Disable Vsync

### What it does

Prevents calls to eglSwapInterval from locking the framerate.  This allows seeing the true framerate of your game rather than, for instance, a max of 60 fps.  On mobile, this option has not been very helpful to me.

### What to do

If vsync is making a significant difference, then you can look to your engine docs to determine whether it can be disabled.  There's always the visual tradeoff here (screen tearing vs slight fps increase) on the PC.  I know that on iOS, you cannot disable vsync.  I am unsure if you can disable vsync on Android, it may be device or vendor specific.

## Null Fragment Shader

### What it does

Replaces all fragment shaders with a simple and fast fragment shader.

### What to do

If your fps improves then your fragment shaders are too complex.  If you are working in a game engine like ShiVa 3D then this means that you are doing a little too much in terms of materials and effects:

*   Reduce or eliminate Post Processing (SSAO, Bloom, Gamma, Color filters, etc).  Or possibly, use offscreen rendering to render post effects at a smaller size to boost post processing speed.  Here's a hint, if enabling null fragment shaders makes your entire screen disappear, it's probably because you are using post-effects  you can actually scrub through the frame scrubber and find a draw call that is blitting this frame to the screen to validate
*   Reduce number of dynamic lights per object.  If you are using per-pixel lighting, you may need to change to per-vertex.
*   Reduce shadow complexity
*   Reduce material complexity  for instance if you have a material animation that uses opaque textures, the cost to calculate each pixel of the material may be high.

## Null Viewport

### What it does

Where Ignore Draw Calls just waits for draw calls and throws them on the floor, Null Viewport still renders everything but at an infinitesimally small size.

### What to do

Drill down into specific tests  Blending, Fragment Shader, 22 Textures.  If none of these are increasing your framerate, look at draw calls, polycount or disabling post-processing effects.

## Disable Blending

### What it does

Prevents all alpha-blended drawing.  Arguably one of the most handy checkboxes.  Not only does it let you see if you are getting hit by overdraw, but it also visually shows you the areas where overdraw is happening in your game.

<a href='{{ site.url }}/assets/uploads/2013/11/blending.jpg'><img src='{{ site.url }}/assets/uploads/2013/11/blending-300x158.jpg' alt='blending'></a>

### What to do

Uhh, get rid of alpha blending!  This could mean:

*   Reducing the emitter count of a heavy particle system
*   Swapping out opaque textures for solid textures
*   Reducing the number of opaque textures that you layer on HUD items
*   Reducing the number of lights hitting the same object (each light must be blended)

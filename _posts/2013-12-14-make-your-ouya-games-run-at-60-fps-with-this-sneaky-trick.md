---
title: Make Your Ouya Games Run at 60 FPS with This Sneaky Trick!
author: error454
layout: post
permalink: /2013/12/14/make-your-ouya-games-run-at-60-fps-with-this-sneaky-trick/
categories:
  - Android
  - Game Theory
  - OUYA
  - Shiva 3D
  - Tegra 3
tags:
  - gpu
  - ouya
  - perfhud
  - shiva
  - tegra
---
<img src='{{ site.url }}/assets/uploads/2013/12/bsantos_Empty_Box_Thinking-copy.png' alt='Once the awesome box has been opened, it cannot be closed...'>

I've been playing around in perfHUD ES **a lot** lately profiling OUYA performance.  One of the cool features it provides is the ability to look at and replace shaders on the fly.  I thought this was a great opportunity to investigate how costly various shaders are.
<!--more-->
If you're only here for the graphs, scroll to the end.  If you'd like a little GPU 101 to help interpret the graphs, then read on.

WARNING! This topic is a total Pandora's box of awesomeness and potential confusion!  I'm going to do my best to help you assimilate some of the industry knowledge behind GPU performance in this article in a way that's hopefully useful to you as a game developer.

If you see any incorrect statements, please leave a comment or hit me on twitter so I can correct it/them! Everything in this article was measured on the Tegra 3 (OUYA), which means Android 4.1.2.  My 3D game engine of choice is <a href="http://www.shivaengine.com/" target="_blank">Shiva Engine</a>, but the knowledge I'm presenting applies to any game engine.

# GPU 101

GPUs are. . . complicated!  Let's look at them from the perspective of a game developer working in a 3D game engine. Forget everything you know about GPU Architecture and let's dive in!

Here's what my game looks like, it's a riveting game consisting of 3 primitives.

<img src='{{ site.url }}/assets/uploads/2013/12/gamescene.png' alt='The game scene'>

## GPU Architecture

Let's use the above scene to break down the architecture of the GPU.

<img src='{{ site.url }}/assets/uploads/2013/12/gpu-architecture.png' alt='GPU Architecture. The sphere has been processed while the other 2 shapes are waiting in line.'>

So, every model in your scene lines up on the conveyor belt, waiting for his very own Draw Call.  When his turn (draw call) comes, he's passed through the vertex shader(s), then the fragment shader(s) and is finally thrown onto the screen (technically not really, but from a high-level view just go with it).  When all the draw calls are finished, your game has officially drawn 1 frame.  Hopefully it does all this 60 times a second, because we all love 60 FPS right?!

## A Bit About Hardware

There are 2 types of GPU Architectures out there.  Fixed function and Unified.  The super important difference between these comes down to how the Vertex Shaders and Fragment Shaders are implemented in hardware.

### Fixed Function

A fixed function architecture is kind of old school.  It designates a fixed number of vertex shaders and a fixed number of fragment shaders.  Fixed means *unchanging, *meaning that vertex shaders can't do anything besides vertex shading. They are stuck in life doing nothing but running vertex programs.  You will find fixed function architectures in mobile devices because they can be made more power efficient.

An example of a fixed function architecture is the Tegra 3 which has:

*   4 Vertex Shaders
*   8 Fragment Shaders

<a href='{{ site.url }}/assets/uploads/2013/12/shaders.png'><img src='{{ site.url }}/assets/uploads/2013/12/shaders-1024x695.png' alt='Tegra 3 Shaders'></a>

### Unified

Unified shaders are what you find on desktop GPUs.  In this architecture, there is a pool of shaders that can be used as either vertex shaders or fragment shaders based on whichever is required.  So if there are a ton of vertices that need processing, the GPU can use all the shaders as vertex shaders.

My laptop's Nvidia GTX 650M has 384 Unified Shaders.  Ever wonder why the divide between Mobile and Desktop gaming performance is so big?  This is why, a ton of unified shaders vs a handful of fixed function shaders.

<img src='{{ site.url }}/assets/uploads/2013/12/unifiedshaders.png' alt='The GTX 650M has 384 Unified Shaders.  They can be used as vertex or fragment shaders.'>

## Shaders

<a href='{{ site.url }}/assets/uploads/2013/12/materials.png'><img src='{{ site.url }}/assets/uploads/2013/12/materials-300x173.png' alt='Each material option generates different vertex and fragment programs'></a>

If you don't write your own shaders then you are most likely working with materials.  You might be setting a texture map, turning on per vertex or per pixel lighting, setting diffuse and ambient colors etc.

In the end, all of these material settings are transformed into 2 programs that run on the GPU.  These 2 programs are then run on the vertex and fragment shaders to produce the desired look of your material.

Some material options will generate vertex and fragment programs that are more expensive than others.  For instance, the fresnel effect is very expensive!  Towards the end of this article there are some helpful charts to help you decide which material options you might want to avoid on mobile devices.

## Vertex Shader

<a href='{{ site.url }}/assets/uploads/2013/12/IMG_9714.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/IMG_9714-200x300.jpg' alt='Clock cycles?!?!? Where&apos;s the beef?'></a>

What does this guy do?  Things. . . magical things!  Even if you don't write your own shaders, you should understand some basics about vertex shaders:

*    A vertex shader is a program that runs on the GPU
*   The program runs once for every vertex in your model
*   A vertex program's runtime is measured in cycles
*   One vertex shader cycle directly corresponds to a single GPU clock cycle
*   All vertex programs for a single draw call are run in parallel if there are enough slots

Huh? Why would you how do I Let's break this down into game developer english!

**A Simple material with no lighting = **12 GPU cycles  
**A Simple material with Per Vertex lighting = **23 GPU cycles

<img src='{{ site.url }}/assets/uploads/2013/12/square.png' alt='The cube has 24 vertices'>

12 GPU cycles means that the vertex shader program for the material takes 12 GPU clock cycles to run.  So, let's take our yellow cube as an example.

The cube has 24 vertices and is using Per Vertex lighting.  This means that for the cube's draw call, we are going to:

1.  Send 24 vertices to the <strong style="font-family: sans-serif; font-size: medium; font-style: normal; font-variant: normal;">Vertex Shader(s)</strong>
2.  For each vertex, a program taking 23 cycles is going to run
3.  The 24 programs will run in parallel.  Obviously if we have only 4 vertex shaders on the GPU, then only 4 programs can run at a time while the remaining programs wait for an empty slot.

#### Whoah Whoah Whoah! Bottleneck Alert?!

I know what you're thinking!  You're looking at the Tegra 3 architecture, you see the 4 little vertex shaders vs the 8 fragment shaders and are thinking, these 4 little dudes are obviously going to be my bottleneck!  Hmm, let's just take a peek at the math and find out!

Sticking with Tegra 3, we look up his <a href="http://en.wikipedia.org/wiki/Tegra#Tegra_3" target="_blank">GPU specs</a> and see that the GPU runs at 520 MHz, cool!  Let's focus our thoughts around a single frame.  We can calculate the number of Vertex Shader clock cycles that are available to us in a single frame at 60 FPS.

<p style="text-align: center;">
  <img src="//s0.wp.com/latex.php?latex=520%5Ccdot+10%5E6%5C%3B+%5Cdfrac%7Bcycles%7D%7Bsec%7D+%5Ccdot+%5Cdfrac%7B1%5C%3B+sec%7D%7B60%5C%3B+frames%7D+%5Ccdot+4%5C%3B+Shaders+%3D+34%2C666%2C666+%5C%3B%5Cdfrac%7Bcycles%7D%7Bframe%7D&#038;bg=ffffff&#038;fg=000&#038;s=2" alt="520&#92;cdot 10^6&#92;; &#92;dfrac{cycles}{sec} &#92;cdot &#92;dfrac{1&#92;; sec}{60&#92;; frames} &#92;cdot 4&#92;; Shaders = 34,666,666 &#92;;&#92;dfrac{cycles}{frame}" title="520&#92;cdot 10^6&#92;; &#92;dfrac{cycles}{sec} &#92;cdot &#92;dfrac{1&#92;; sec}{60&#92;; frames} &#92;cdot 4&#92;; Shaders = 34,666,666 &#92;;&#92;dfrac{cycles}{frame}" class="latex" /></img>
</p>

OK, let's make this even more useful.  Let's say we've decided we want to use Per Vertex lighting on all our objects. Knowing that Per Vertex lighting costs 23 cycles, we can actually calculate a ballpark for the maximum number of vertices per frame.

<p style="text-align: center;">
  <img src="//s0.wp.com/latex.php?latex=34%2C666%2C666+%5C%3B%5Cdfrac%7Bcycles%7D%7Bframe%7D%5Ccdot+%5Cdfrac%7B1+%5C%3Bvertex%7D%7B23+%5C%3Bcycles%7D+%3D+1%2C507%2C246+%5C%3B+%5Cdfrac%7Bvertices%7D%7Bframe%7D&#038;bg=ffffff&#038;fg=000&#038;s=2" alt="34,666,666 &#92;;&#92;dfrac{cycles}{frame}&#92;cdot &#92;dfrac{1 &#92;;vertex}{23 &#92;;cycles} = 1,507,246 &#92;; &#92;dfrac{vertices}{frame}" title="34,666,666 &#92;;&#92;dfrac{cycles}{frame}&#92;cdot &#92;dfrac{1 &#92;;vertex}{23 &#92;;cycles} = 1,507,246 &#92;; &#92;dfrac{vertices}{frame}" class="latex" /></img>
</p>

<p style="text-align: left;">
  Santa came early this year because that's a ton of vertices!  Just looking at the number of vertex shaders alone can be quite deceiving as you can see.  Now, these calculations involve a little bit of hand-waving because things just don't quite work like this, see the reality check section below.  However, this still serves as a pretty good baseline for things.
</p>

<p style="text-align: left;">
  I would be negligent not to mention the official recommendations by nvidia for Tegra 3.  <a href="http://docs.nvidia.com/tegra/Content/PHES_Analyze_Geometry.html" target="_blank">Peak throughput is achieved at 10 cycles per vertex</a>.
</p>

## Fragment Shader (also called pixel shaders)

Like vertex shaders, fragment shaders are just another program that runs on the GPU.  Here are their highlights.

*   A fragment shader is a program that runs on the GPU
*   The program runs once for every fragment generated by your model
*   At least 1 fragment is generated for every pixel of your model, more complicated materials may generate more fragments
*   One fragment shader cycle directly corresponds to a single GPU clock cycle
*   All fragment programs for a single draw call are run in parallel if there are enough slots

<img src='{{ site.url }}/assets/uploads/2013/12/fragments.png' alt='A fake example showing 25 fragments.'>

The main thing to understand about fragment shaders is the number of fragment programs that are generated.  In most cases, you are talking 1 fragment per pixel.

To the right you can see a faked example.  Basically, every pixel that is occupied on the screen is turned into a fragment.  In reality, this cube is about 120 x 120 pixels and would therefore generate 120 * 120 = 14,400 fragments.  What's most important to remember is:

1.  The larger a model is on screen, the more fragments it will generate
2.  More complex materials require more cycles

Knowing this, you can see how you might want to be wary of materials that eat lots of fragment shader cycles, especially if they occupy a large portion of the screen!

### Bottleneck Check

Let's do some off the cuff math to estimate the power of our fragment shaders on the Tegra 3.  First, how many pixel shader cycles do we have per frame? Twice as many as we do fragment cycles.

<p style="text-align: center;">
  <img src="//s0.wp.com/latex.php?latex=520%5Ccdot+10%5E6%5C%3B+%5Cdfrac%7Bcycles%7D%7Bsec%7D+%5Ccdot+%5Cdfrac%7B1%5C%3B+sec%7D%7B60%5C%3B+frames%7D+%5Ccdot+8%5C%3B+Shaders+%3D+69%2C333%2C333+%5C%3B%5Cdfrac%7Bcycles%7D%7Bframe%7D&#038;bg=ffffff&#038;fg=000&#038;s=2" alt="520&#92;cdot 10^6&#92;; &#92;dfrac{cycles}{sec} &#92;cdot &#92;dfrac{1&#92;; sec}{60&#92;; frames} &#92;cdot 8&#92;; Shaders = 69,333,333 &#92;;&#92;dfrac{cycles}{frame}" title="520&#92;cdot 10^6&#92;; &#92;dfrac{cycles}{sec} &#92;cdot &#92;dfrac{1&#92;; sec}{60&#92;; frames} &#92;cdot 8&#92;; Shaders = 69,333,333 &#92;;&#92;dfrac{cycles}{frame}" class="latex" /></img>
</p>

<p style="text-align: left;">
  How many full 1080p frames can we render if each fragment program were 10 cycles?
</p>

<p style="text-align: left;">
  Well, how many cycles does it take to calculate a single 1080p frame using a material that costs 10 cycles?
</p>

<p style="text-align: center;">
  <img src="//s0.wp.com/latex.php?latex=10%5Ccdot+1920%5Ccdot+1080%5C%3B+cycles+%3D+20%2C736%2C000%5C%3B+cycles&#038;bg=ffffff&#038;fg=000&#038;s=2" alt="10&#92;cdot 1920&#92;cdot 1080&#92;; cycles = 20,736,000&#92;; cycles" title="10&#92;cdot 1920&#92;cdot 1080&#92;; cycles = 20,736,000&#92;; cycles" class="latex" /></img>
</p>

<p style="text-align: left;">
  OK, how many of those can we do per frame?
</p>

<p style="text-align: center;">
  <img src="//s0.wp.com/latex.php?latex=69%2C333%2C333%5C%3B+%5Cdfrac%7Bcycles%7D%7Bframe%7D+%5Ccdot+%5Cdfrac%7B1%5C%3B+frame%7D%7B20%2C736%2C000%5C%3B+cycles%7D%3D3&#038;bg=ffffff&#038;fg=000&#038;s=2" alt="69,333,333&#92;; &#92;dfrac{cycles}{frame} &#92;cdot &#92;dfrac{1&#92;; frame}{20,736,000&#92;; cycles}=3" title="69,333,333&#92;; &#92;dfrac{cycles}{frame} &#92;cdot &#92;dfrac{1&#92;; frame}{20,736,000&#92;; cycles}=3" class="latex" /></img>
</p>

<p style="text-align: left;">
  Again, a bit of hand-waving going on, but the main point here is that at 1080p, the fragment shaders will almost always be your bottleneck.  You can see how post-processing is basically out of the question at full frame resolution since enabling bloom, gamma and contrast filtering would consume all your cycles.
</p>

<p style="text-align: left;">
  Now you can see why using off-screen rendering can give a huge boost to your game.  Your game can render out at 720p and then upscale to 1080p like the <a href="http://www.ign.com/wikis/xbox-one/PS4_vs._Xbox_One_Native_Resolutions_and_Framerates" target="_blank">big boys</a>.
</p>

## Reality Check

For all of my efforts above where I try to give some ballpark numbers, you should know the truth.  The numbers are a lie (very much like the cake).  You see, the numbers I gave would only ever be valid if you were drawing a single piece of geometry with a really snazzy material.

Most of our games consist of quite a bit more than a single draw call.  Here's the rest of the story.  Before the next draw call can happen, the GPU has to wait for the fragment shaders to finish processing the previous draw call.  So something like this:

Draw Call 1: Vertex Shader -> Fragment Shader -> Done  
Draw Call 2: Vertex Shader -> Fragment Shader -> Done  
Draw Call 3: Vertex Shader -> Fragment Shader -> Done

Think about what this might mean in terms of performance.  Let's say you were drawing  50 cubes with all the bells and whistles, real-time lighting, bloom, camera blur, gamma adjustment, contrast filter, SSAO.  In this situation, your vertex shaders will be twiddling their thumbs while the fragment shaders get rocked.  In fact, you could increase the geometry of your cubes by a TON and your performance would stay exactly the same!

This is an important concept to remember, because if you know that your bottleneck is the fragment shaders you can often get some free visual improvements by feeding starving vertex shaders.

# Putting it all together

Thanks to nvidia perfHUD ES, I was able to look at the cost of shaders for Shiva 3D.  It's pretty darn simple, perfHUD shows you a list of all your shaders and which draw call they are associated with.  You can click on a shader and see how many vertex and fragment cycles it consumes.
<a href='{{ site.url }}/assets/uploads/2013/11/shaders.png'><img src='{{ site.url }}/assets/uploads/2013/11/shaders-300x223.png' alt='Shaders'></a>

Below I present the results of my findings organized by lighting and material type.  I also include a special case for the fresnel effect. Because fresnel can be applied to any material, I show the results without fresnel (Normal) and with.

## No Lighting

<a href='{{ site.url }}/assets/uploads/2013/11/no-lighting.png'><img src='{{ site.url }}/assets/uploads/2013/11/no-lighting-1024x642.png' alt='The cost of shaders with no lighting.'></a>

## Vertex Lighting

There is a special caveat with fresnel based lighting here.  When using the fresnel effect with vertex based lighting, you incur 1 extra draw call on your material.  The fresnel numbers below are arrived at by adding the vertex and fragment cycles of both draw calls.

<a href='{{ site.url }}/assets/uploads/2013/11/vertex-lighting.png'><img src='{{ site.url }}/assets/uploads/2013/11/vertex-lighting-1024x637.png' alt='The cost of shaders with vertex lighting'></a>

## Pixel Lighting

I had to run the fresnel test several times to double check these results.  You will note that with fresnel enabled for some material types that you incur less fragment cycles!  I'm only reporting the numbers here, I don't have a great explanation for them.

It could be that the difference in cost is due to the fresnel affect visually dominating what we see, for instance think of the fresnel effect as the Sun and the original material as a flashlight.  No point in turning a flashlight on when it's competing with the sun, right?  On the other hand, why isn't this true for the vertex/no light cases?  PURE speculation here that could be resolved if I went back and stepped through both shaders to see what operations they're performing.

<a href='{{ site.url }}/assets/uploads/2013/11/pixel-lighting.png'><img src='{{ site.url }}/assets/uploads/2013/11/pixel-lighting-1024x644.png' alt='The cost of shaders with pixel lighting.'></a>

That's all!  If you found this useful, please leave a comment!
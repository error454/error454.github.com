---
title: 'Shiva 3D  Benchmarking for Mobile Performance'
author: error454
layout: post
permalink: /2011/02/18/shiva-3d-benchmarking-for-mobile-performance/
categories:
  - Shiva 3D
  - WebOS
tags:
  - benchmark
  - palm
  - shiva
  - vertex count
---
As I was meeting with my indie game team to discuss progress on a project, the topic of 3D models and performance came up.  How many vertices should our models have?  This seemed to be an incredibly valid question to which I had no answer.

The following is how we determined our vertex budget.

## Starting a Vertex Budget

### Step 1. Assign ratios to your categories.

We knew that our scenes would consist of Models, Scenery and Particles.  We assigned our budget as follows:

*   30% Models
*   65% Scenery
*   5% SFX

### Step 2. Benchmark the low-end target

The Palm Pre was the low-end device that we wanted to target.  If we could find the number of vertices that this device could handle while maintaining 30 fps, we'd have all we need.  I wrote a simple benchmark in Shiva to do this, see below if you'd like to download the package.

I primarily wanted to see the effects that texture size and realtime lighting had on our performance.  I'll possibly add shadows in the future.

### Step 3. Interpret Results

I ran my benchmark on an EVO 4G along with my overclocked Palm Pre Plus.  With no textures or lighting, the EVO 4G was pulling around 6000 vertices where the Pre was clocking in at 9000?!?  I scratched my head for a second before realizing that the EVO was running at a much higher screen resolution.

Here are the graphs from my Palm Pre runs.

<a href=''><img src='{{ site.url }}/assets/uploads/2011/02/no-lighting.png' alt=''></a>

<a href=''><img src='{{ site.url }}/assets/uploads/2011/02/lighting.png' alt=''></a>

Armed with these numbers, we calculated our final values using 6000 vertices as a target to give ourselves a little headroom:

*   30% Models
    * 6000 vertices * 30% / 6 models on screen = 300 vertices per model
  
* 65% Scenery 
    * 6000 * 0.65 = 3900
  
* 5% SFX
    * 6000 * 0.05 = 300

##Get The Benchmark

UPDATE: There has been some <a href="http://www.stonetrip.com/developer/forum/viewtopic.php?f=25&t=22369" target="_blank">discussion about this benchmark</a> over on the Shiva forums.  Some points mentioned were:

* The log..*() function kills performance and needs to be removed.
* Multiple textures would be more realistic
* Multiple lights desired
* Ability to choose light mode

To disclaim some of my poor variable names, I have to say this benchmark embraces the essence of late-night feverish coding.  Also, as of now, the Shiva UI isn't very forgiving about renaming functions and handlers so forgive me for not planning more.

I welcome suggestions and considerations for this project, please let me know if you see something I could do better. Hopefully this will give you a starting point for your own vertex budget.

<a title="Shiva Benchmark" href="http://bit.ly/fx2Izy">Shiva Benchmark</a>

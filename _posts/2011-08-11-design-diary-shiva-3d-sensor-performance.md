---
title: 'Design Diary  Shiva 3D Sensor Performance'
author: error454
layout: post
permalink: /2011/08/11/design-diary-shiva-3d-sensor-performance/
categories:
  - Shiva 3D
tags:
  - fps
  - onSensorCollision
  - onSensorCollisionBegin
  - performance
  - sensors
  - shiva
  - shiva3d
---
The use of the proper sensor in Shiva is critical to obtaining optimal frame-rate. In the past, you may have hap-hazardly populated onSensorCollision without a second thought. This could cause serious performance issues down the road.

The largest difference in the 3 sensor handlers are the circumstances under which they are fired. I'm sure you are familiar with these differences already, but if you aren't here is a quick recap.

*   **onSensorCollision**  Called on every frame as long as sensors are colliding.
*   **onSensorCollisionBegin**  Called every frame that sensors are colliding as long as they weren't colliding the previous frame.
*   **onSensorCollisionEnd**  Called every frame that sensors are not colliding as long as they were colliding the previous frame.
<!--more-->
You may be thinking, *yeah dude I already know all of this* but what you might not know is the minimum performance impact that these sensor handlers impose.

  
# The Benchmark

I devised the following benchmark to measure this minimum performance impact. This benchmark can easily be reproduced by anyone, the steps to do so are as easy as 1-2-9:

1.  Create a blank project
2.  Create an empty scene with a camera
3.  Create a new shape (box)
4.  Create a new empty object AI
5.  Create a new empty user AI
6.  Create a HUD containing a label to hold the FPS
7.  In the user AI onEnterFrame, print the FPS
8.  In the user AI onMouseButtonUp handler, create a runtime object and print the current count of objects
9.  Count the number of objects that can exist while maintaining 30 FPS

Because objects spawn in the center of the scene, all sensors will overlap. This results in the sensor handlers being called (if applicable). Below are the resulting data sets from these tests.

# The Tests

I compared box vs spherical sensors, sensor quantity and onSensorCollision vs onSensorCollisionBegin/End. I ran these tests inside the editor with the preview mode set to Runtime (no debug objects being shown). This was ran on my 1.2 GHz netbook.

## The Baseline

First I wanted to establish a baseline for sensor performance. This test was ran without any sensor handlers attached to the object. The performance can't get any better than this.

<img src='{{ site.url }}/assets/uploads/2011/08/chart_1.png' alt=''>

<p style="text-align:left;">
  The baseline indicated that box sensors were less performant than spherical sensors (head scratch).
</p>

## onSensorCollision

I then added an onSensorCollision handler and repeated the tests while varying the sensor count. Keep in mind that my onSensorCollision is doing \*absolutely nothing\*, it literally doesn't have any code in it. The extreme performance difference here is due to the overhead involved in the sensors.

<img src='{{ site.url }}/assets/uploads/2011/08/chart_2.png' alt=''>

## onSensorCollisionBegin

I then switched collision handlers to onSensorCollisionBegin and ran the same tests as above.

<img src='{{ site.url }}/assets/uploads/2011/08/chart_3.png' alt=''>

Whoah! Huge difference! Clearly the reason why is that the sensor collision only hits the CPU a single time (when the sensors first collide) vs every single frame. These numbers are very close to the baseline, on average being ~15 objects off. There is still a bit of overhead, but clearly far better performance than the frame-by-frame situation.

# Takeaway

The main items to take away from this data are:

*   Spherical sensors seem to be more performant than box sensors
*   Don't blindly use onSensorCollision unless you actually need collision logic throughout the entirety of a collision.
*   Using onSensorCollisionBegin/End will minimize the CPU impact and allow for a higher object count in the scene.
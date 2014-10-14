---
title: 'Designer Diary: Shiva AI Navigation  Dynamics vs Translation'
author: error454
layout: post
permalink: /2011/07/04/designer-diary-shiva-ai-navigation-dynamics-vs-translation/
categories:
  - Shiva 3D
tags:
  - ai
  - avoid
  - dynamics
  - flee
  - navigation
  - offsetpursuit
  - pursuit
  - seek
  - shiva
  - shiva3d
  - translation
---
<img src='' alt=''>

## AI Navigation  Dynamics vs Translation

<img src='' alt=''>

<a href=''><img src='{{ site.url }}/assets/uploads/2011/07/level0-spawn-points.jpg' alt=''></a>
<a href=''><img src='{{ site.url }}/assets/uploads/2011/07/level0-spawn-points.jpg' alt=''></a>



We got to the point where we wanted to start moving the mothership around, again using a setTranslation model meant that we had to figure out equations to get it to a specific location on screen.  We did learn a lot about splines and I was happy that Shiva accomodated us on our fools errand by providing functions to evaluate BSpline, Bezier and CatmullRom.

I call this translation model the Infinite Momentum Model because we were trying to use the dynamics system for collisions, but as you can guess, setting a translation every frame completely breaks the collision system.  For collisions to work, the dynamics system needs the opportunity to allow the results of the collision to take place. By setting translation every frame, you are overriding the result of the collision as if your object had infinite momentum.

In game, this manifested itself by drone ships colliding with and pushing planets off the screen. . . Our model was clearly broken for a few reasons:

*   We spent more time arguing over the correct pronunciation of Bezier than we spent implementing it
*   We were unable to use the collisions system
*   Each navigation destination required hand-calculated math and difficult spline creation for arcs
*   Overall the system was inflexible

## New AI Implementation

The new implementation was discovered by <a href="http://gamedev.stackexchange.com/questions/8045/ai-control-for-a-ship-with-physics-model" target="_blank">a question on the game development stackexchange site</a>.  One of the answers linked to a paper by Craig W. Reynolds titled <a href="http://red3d.com/cwr/steer/gdc99/" target="_blank">Steering Behaviors For Autonomous Characters</a>.  For anyone looking how to implement AI steering, this is the gold mine.  The AI behaviors implemented from this paper were:

* Arrive
* Avoid
* Seek
* Flee
* Evade
* Offset Pursuit
* Pursuit
  
We also implemented some higher-level behaviors that simply mix the lower-level behaviors together, like OffsetPursueFireAvoid to do a shooting flyby.  In this video you can see an example of the Arrive and Pursuit behaviors.  The Red ship is being controller by a mouse click while the green ship is pursuing.

  <a href=''><img src='{{ site.url }}/assets/uploads/2011/07/level0-spawn-points.jpg' alt=''></a>

This video is an example of OffsetPursueFireAvoid.  Also at the end, you can see an example of OffsetPursuit being used to create a formation of drones.



### Shiva Specific Implementation of AI Steering

I wanted all AI behaviors to have a self-preservation priority, meaning they don't just crash directly into the planet if it lies in their path.  To accomplish this we used the following basic flowchart.

<img src='{{ site.url }}/assets/uploads/2011/07/ai-flowchart.png' alt=''>

One disadvantage to this behavioral flow is that you get bouncies.  Sometimes when close to an obstacle, the drone will bounce between Avoid and another AI behavior quite rapidly.  I thought there might be an advantage to building some hysteresis into this flowchart but that is on the back-burner.

<img src='{{ site.url }}/assets/uploads/2011/07/ai-flowchart.png' alt=''>

You'll notice in the first video above that I begin in debug mode and that you can see the sensors attached to the ship.  Our drone ship has 2 sensors with unique sensor IDs

* A body sensor
* A long navigation sensor

The navigation sensor is what protrudes from the front of the ship and alerts the ship of an impending collision.  Right now, we have the drones avoid only the planet, but we could easily add other things for them to avoid (like each other).  The length of the navigation sensor is supposed to be a function of the max velocity of the object.  We didn't implement this dynamic sizing because modifying the sensor size and offset programatically sounded problematic.  Instead, to determine the required length of the sensor we followed a 3-step process.

Before looking at the process, it is important that your object has mass set correctly in the dynamics controller. The default mass is something like 75.  Because we are dealing with planets and motherships, we setup our mass scale like this:

* drone laser	1
* drone		5
* rocket		10
* sat		50
* mothership	1000
* planet		9999
  
The process to determine the required sensor length is then:
* Choose the maximum velocity of the drone.  This is based on what feels right, too fast, too slow, just right.
* Choose the maximum turning force of the drone.  Again based on how good it feels, the drone was either turning in too large of arcs or was turning far too quickly.
* Set the sensor length so that the drone has time to avoid collision when approaching a planet from dead center at maximum velocity.

If I ever have to go back and tweak this, I might just take a try at the dynamic sizing model.

 <img src='{{ site.url }}/assets/uploads/2011/07/ai-flowchart.png' alt=''>
    
I am incredibly pleased with how simple this AI model is to use.  I have overloaded handlers in the model so that I can simply say onSeek(this.someTarget) or onSeek(x, z).
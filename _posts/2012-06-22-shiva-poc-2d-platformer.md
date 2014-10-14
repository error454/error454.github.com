---
title: 'ShiVa PoC: 2D Platformer'
author: error454
layout: post
permalink: /2012/06/22/shiva-poc-2d-platformer/
categories:
  - Shiva 3D
tags:
  - 2d
  - jumpman
  - platform
  - platformer
  - poc
  - shiva
---
I finally got around to a little proof of concept for a 2D platform type game.  This PoC is really part tool and part test.  The idea was to test out a design for a 2D platformer that uses the dynamics engine and to make a tool to tweak  physics parameters to dial-in the correct feel.  In the end I don't think I'll go forward with this particular implementation.  This project has brought to mind several issues that one might encounter when using the dynamics system for platformer physics.

<a href='https://dl.dropbox.com/u/7079101/shiva/jumpMan.html'><img src='{{ site.url }}/assets/uploads/2012/06/jumpman.jpg' alt=''></a>

I now think that using the dynamics system for a serious 2D platformer is probably the wrong approach.  The first problem is that you don't have fine-grain control over jump-mechanics.   For instance it isn't possible to easily define a way to make your hero  jump exactly 3 game units high. I still haven't entirely solved the (working perfectly/not working at all)  <a href="http://www.stonetrip.com/developer/forum/viewtopic.php?f=13&t=26018" target="_blank">moving platform problem</a> either.



## The Problem With Jumping

In the jump state, I wanted to allow the hero to jump higher by holding the jump button longer.  This works, but only at high frame-rates.  The problem is a combination of 2 things:

1.  The dynamics engine has its own timestep that is independent of framerate
2.  The jump state can only be controlled once every frame

The jump AI works like this:  
Apply impulse -> Wait for X seconds and then set vertical impulse to zero -> Apply additional upforce for the next Y seconds if the user is holding the button ->Apply gravity

So you can imagine that if framerate gets low, the logic that controls when to start/stop different jump stages is delayed.  At the same time, the dynamics engine is still moving things along separate from the frame calculation.  The end result is that at lower framerates, the character ends up jumping higher because the jump logic isn't running as often.

It's not that the dynamics system is bad and it's not that there's a problem with the engine.  The problem is that the mechanism used to update the translation of the hero is decoupled from the mechanism used to control the hero state.  Therefore to really gain control of the situation, we'll need to implement our own physics system that translates the character in a frame-independent way.

## The Ideal System

The ideal  platformer (yet another PoC in the works here) would let you define:

*   Hero jump height in units of Hero tallness.  i.e. 3 units means the hero jumps 3x his height
*   Jump acceleration
*   Fall acceleration

Hero jump height is critical if you're designing a solid paltformer world.  The level designer needs to know exactly how many units high and far the hero can jump.  You might still be thinking, hmm I *could* break out a couple physics story problems to solve this with dynamics:

*   What velocity **v** should be applied to an object for it to reach height **h**?
*   What impulse **j** is needed to achieve the velocity **v** in the time interval **dt**?

I think at this point we're just fighting against the dynamics system, and also I think you'd go crazy trying to find the right time step and iteration count to make the dynamics engine accuracy meet your calculated values.

Depending on the type of game you are making, the behavior I've created here may be good enough.  Please let me know if you've used other approaches!

<a href="https://github.com/error454/ShiVa-Proof-Of-Concept" target="_blank">Get the source</a>
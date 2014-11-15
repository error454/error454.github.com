---
title: 'ShiVa PoC: Box Particle Lighting'
author: error454
layout: post
permalink: /2012/06/02/shiva-poc-box-particle-lighting/
publicize_results:
  - 'a:1:{s:7:"twitter";a:1:{i:37590404;a:2:{s:7:"user_id";s:8:"error454";s:7:"post_id";s:18:"209032941559480320";}}}'
categories:
  - Shiva 3D
tags:
  - dynamics
  - lighting
  - particles
  - shiva
  - tutorial
---
I have started a new <a href="https://github.com/error454/ShiVa-Proof-Of-Concept" target="_blank">code repository on github</a> where I will be placing various self-contained Proof of Concept projects for the ShiVa game engine.  Each project will be contained in a separate directory for easy dissemination.  I will also include live web-based samples of all projects here on my blog as I create them.  This is the first of those projects, click the image to load the live web version.

<a href='https://dl.dropbox.com/u/7079101/shiva/boxParticleLighting.html'><img src='{{ site.url }}/assets/uploads/2012/06/boxparticle.jpg' alt=''></a>

This demo is meant to test particle interaction with colliders along with a moving dynamic light source.
<!--more-->


## Scene Setup

The project scene is a simple set of planes, set as colliders.  These planes form a fully enclosed cube so that the object inside cannot escape.  The helper object has a point-light source, a particle emitter and a dynamics controller.  Every frame, a random impulse is added to the dynamics controller of the helper to give it motion.  The dynamics controller has gravity disabled but dynamics and collisions enabled.

## What to Tweak

*   **Particles ** This is a great project to get a feel for particle system parameters, edit them while you run the demo.  One issue with particle systems is that we usually tweak the parameters while the particle emitter is static.  The particle system often looks completely different once the emitter is in motion, so tweaking while the emitter is static can be quite deceiving!
*   **Light ** The radius of a point light source may be difficult to get right, this is a good environment to get a feel for the effect of point light source scaling as there aren't any other light sources.
*   **Dynamics **Try changing the mass or maximum velocity of the emitter objects dynamics controller to get a feel for the wonderful dynamics that ShiVa provides.  Also try changing the random values generated in onEnterFrame that determine how large of an impulse to add to the object.

## Applications

You might use this to simulate a firefly stuck in a jar, or an object with a chaotic trajectory.  The code is less than a dozen lines, the project is small and to the point, feel free to check it out and use it in your projects.

<a href="https://github.com/error454/ShiVa-Proof-Of-Concept" target="_blank">Grab the source</a>
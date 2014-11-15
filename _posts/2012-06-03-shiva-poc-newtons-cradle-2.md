---
title: 'ShiVa PoC: Newtons Cradle'
author: error454
layout: post
permalink: /2012/06/03/shiva-poc-newtons-cradle-2/
categories:
  - Shiva 3D
tags:
  - ball joint
  - cradle
  - dynamics
  - joints
  - newton
  - shiva
---

This is an attempt to replicate a Newton's Cradle device.

<a href='https://dl.dropbox.com/u/7079101/shiva/newtonsCradle.html'><img src='{{ site.url }}/assets/uploads/2012/06/newtons.jpg?w=487' alt='a newton&apos;s cradle'></a>
<!--more-->


This is very similar to the rope sample provided by Stonetrip.  The ball bearings have spherical dynamics bodies and are attached to an anchor point on the ceiling.  The string is cosmetic only, it has no dynamics or anchor points.

The collision doesn't happen like the real world counterpart despite my fiddling with dynamics parameters.  I believe a programmatic solution would not be difficult by simply detecting the last collision point and setting things to be stationary.  I didn't go through the trouble with the programmatic solution since the goal was to see whether the dynamics system would handle the system automatically.

<a href="https://github.com/error454/ShiVa-Proof-Of-Concept" target="_blank">Get the source.</a>


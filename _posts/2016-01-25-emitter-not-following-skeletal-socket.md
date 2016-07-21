---
title: Emitter Not Following Skeletal Socket
author: error454
layout: post
permalink: /2016/01/25/emitter/not/following/skeletal/socket/
categories:
  - ue4
tags:
  - lab notebook
  - ue4
  - physics
  - particle emitter
  - offset
  - latency
  - lag
  - physx
  - beam
  - skeletal socket
  - transform
  - component
  
---
There’s a major bug in our project that is exhibited by beam particles that are not correctly emitting from the socket location that the emitter is attached to. The problem has been seen in every version of UE4 going back to 4.5. Although the problem is exhibited using beam particles, my guess is that it has nothing to do with beam particles and more to do with the entire skeletal transform being a frame behind.

The current way to produce the issue is:

1. Attach a skeletal mesh to a physics simulating body
2. Attach a particle system to some socket in the skeletal mesh
2. Move the physics body around
3. Observe that beam emitters are lagging behind the skeletal transform

<!--more-->

# Reproduction #

The first step is to get a solid minimal reproduction of the issue. Our reproduction should try to address the following:

* Can we produce the problem just by using debug lines (this will eliminate beam emitters from the equation)?
* Can we produce the problem just by simulating physics directly on a skeletal mesh (this will eliminate parent/child hierarchies from the equation)?
* Can we resolve the issue by changing the tick group?

The [reproduction project](https://dl.dropboxusercontent.com/u/7079101/ue4/UE4-SkeletalSocketEmitterLag.7z) is archived for posterity.

# Reproduction Results #

We weren't able to create a reproduction that allowed us to simulate physics directly on the skeletal mesh. Things get complicated with skeletal mesh physics since it’s not a single rigid body in the simulation. We quickly abandoned that approach and fell back to the simulation of a single rigid body with the skeletal mesh attached to it.

<img src='{{ site.url }}/assets/uploads/2016/01/test_bp.jpg'>

We then added significant forces to the rigid body to get it to move in an oscillating pattern. Finally we attached a camera to the skeleton that was zoomed in on the emitter attachment point so that we could easily observe the alignment of the emitter and/or the drawn debug line.

The results of the reproduction can be seen in this screenshot.

<img src='{{ site.url }}/assets/uploads/2016/01/repro.jpg'>

We were able to reproduce the issue using only debug lines and we were able to resolve the issue just by changing the tick group. Both the Post-Physics and Post Update Work tick groups work as desired, the code to draw the actual debug line is handled in the assigned tick group (rather than in the level tick for instance).

Extending the reproduction by attaching a beam emitter, we can see that things continue to work as desired in the post physics/post update work cases.

<img src='{{ site.url }}/assets/uploads/2016/01/emitter_repro.jpg'>

# Lessons Learned #

When a skeletal mesh is attached to a physics object or is driven by physics, the socket transforms can lag behind the physics simulation. Depending on your needs, you can overcome the problem a few different ways. 

If you only need attached sub-objects (like a particle emitter) to stay in the correct location, i.e. attached to a socket, then change the tick group of the skeletal mesh that they're attached to so that it is post physics.

If you need to programatically access the transform of a socket, make sure that any work you do occurs in a tick group that occurs post-physics.



---
title: 'ShiVa 1.9.2  New API Calls'
author: error454
layout: post
permalink: /2013/12/24/shiva-1-9-2-new-api-calls/
categories:
  - Shiva 3D
tags:
  - api
  - changes
  - shiva
---
With the newly released <a href="http://www.shivaengine.com/what-is-shiva-3d.html" target="_blank">ShiVa 1.9.2 Game Engine</a> came a bunch of new API calls.  Not all of these calls have been documented, until now!  Head over to my <a href="https://github.com/error454/Shiva-Missing-Docs" target="_blank">github repo</a> for a comprehensive list of new API calls that showed up between 1.9.1 and 1.9.2.

The biggest highlights in my mind are.

## Average Frame Time

<pre>application.setUseAverageFrameTime ( bUse )</pre>

This should be a default.  It makes all the internal engine calls use average frame time.  This could mean smoothing out jitters in the dynamics system and is something everyone should try.

## Collider Create/Destroy

<pre>collider.create ( hObject )
collider.destroy ( hObject )</pre>

Being able to create and destroy colliders at runtime solves many problems, particularly if your meshes are being combined.

## Offscreen Output, Clipping Children, List Item Children

Offscreen output lets you easily render a HUD to a rendermap.

<pre>hud.enableOffscreenOutput ( hUser, sRenderMap, bEnabled )</pre>

Enabling children clipping will clip children that spill out of their parent container.

<pre>hud.setContainerClipChildren ( hComponent, bClip )</pre>

List item children allows you to use a HUD template inside of a list!

<pre>hud.setListItemChildAt ( hComponent, nItem, nColumn, hChild )</pre>

## Disabling Logging

Possibly one of my absolute favorites!  Shuts down those noisy logs:

<pre>if system.getClientType ( ) == system.kClientTypeEditor then
    log.enable ( false )
end</pre>

## Toggling AI Models

Now instead of adding/removing AI models, you can disable them. This provides a very clean solution for toggling sets of functionality!

<pre>object.enableAIModel ( hObject, sAIModel, bEnable )
user.enableAIModel ( hUser, sAIModel, bEnable )</pre>

## Attractors, Vortex Fields and Turbulence

The particle system gets better with every release!  There are simply too many new functions here.  Here's a sample that creates a vortex field.

<pre>local hBox = application.getCurrentUserSceneTaggedObject ( "box" )
sfx.addParticleVortexField ( hBox )
sfx.setParticleVortexFieldAxialDrop ( hBox, 0, 0.1 )
sfx.setParticleVortexFieldAxialDropDamping ( hBox, 0, 0 )
sfx.setParticleVortexFieldOrbitalSpeed ( hBox, 0, 1 )
sfx.setParticleVortexFieldPosition ( hBox, 0, 0, 0, 0, object.kLocalSpace )
sfx.setParticleVortexFieldRadialPull ( hBox, 0, 3 )
sfx.setParticleVortexFieldRadialPullDamping ( hBox, 0, 0 )
sfx.setParticleVortexFieldStrength ( hBox, 0, 1 )</pre>

## Everything Else

You'll find the rest of the API changes on my <a href="https://github.com/error454/Shiva-Missing-Docs" target="_blank">github page</a>, feel free to submit changes or samples.

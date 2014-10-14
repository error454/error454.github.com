---
title: 'ShiVa 3D  Making a 2D Laser'
author: error454
layout: post
permalink: /2011/04/23/shiva-3d-making-a-2d-laser-attempt-1/
categories:
  - Shiva 3D
tags:
  - 2d
  - effect
  - laser
  - lazer
  - shiva
  - shiva 3d
  - shiva3d
---
<a href='{{ site.url }}/assets/uploads/2011/04/pewpew.png'><img src='{{ site.url }}/assets/uploads/2011/04/pewpew.png?w=300' alt=''></a>

We needed a laser for our WIP space schmup.  This solution seemed to be the most obvious way to create a variable length laser.  I am hoping someone will post a comment telling me how un-optimized it is so that I can improve it <img src="http://www.error454.com/wp-includes/images/smilies/icon_wink.gif" alt=";)" class="wp-smiley" /> 

## How to Make a Laser

**Step 1**. Model and texture a cube whose origin is the center of an edge face.

<a href='{{ site.url }}/assets/uploads/2011/04/postit.jpg'><img src='{{ site.url }}/assets/uploads/2011/04/postit.jpg?w=300' alt=''></a>

Above is the reference design I gave to my artist for this simple design.  It is important that the origin be on the edge to simplify the math involved.  It is unfortunate that ShiVa does not allow you to change the origin of shapes in the model view as it would have been trivial to make a cube and slap a texture on it.

<a href='{{ site.url }}/assets/uploads/2011/04/laser-cube.png'><img src='{{ site.url }}/assets/uploads/2011/04/laser-cube.png?w=300' alt=''></a>

**Step 2**. Scale the cube so that the length of the laser has a scale of 1.

After applying a scale of (0.3, 0.001, 1), you can probably figure out how this is going to work.

<a href='{{ site.url }}/assets/uploads/2011/04/laser-flattened.png'><img src='{{ site.url }}/assets/uploads/2011/04/laser-flattened.png?w=300' alt=''></a>

**Step 3**. Program the laser.

The basic structure looks like the below photo.  The main userAI passes mouse coordinates (nPointX, nPointY) to the laserAI when the user clicks or moves the mouse/their finger.

<a href='{{ site.url }}/assets/uploads/2011/04/ai-map.jpg'><img src='{{ site.url }}/assets/uploads/2011/04/ai-map.jpg?w=300' alt=''></a>

We will explore the laserAI implementation below, I have crossed out some items that are specific to my implementation so that things don't get overly complicated.

<a href='{{ site.url }}/assets/uploads/2011/04/laserai.png'><img src='{{ site.url }}/assets/uploads/2011/04/laserai.png?w=274' alt=''></a>

## The Variables

*   **hSat** An object that will be firing the laser.  In my case it is a satellite, hence the name hSat.  The reason we need this object reference is so that we can position the laser in the scene.  This could just as easily be some fixed point in your scene if your camera never moves.
*   **mbFiring** A boolean that tracks whether the laser is firing.

## The Handlers

**onFireLaser(x, y)**

This is so simple, I don't think I need to explain it.  Set our boolean to true and call the fireLaser function passing the x and y coordinates in.

<pre>--------------------------------------------------------------------------------
function laserAI.onFireLaser ( x, y )
--------------------------------------------------------------------------------

	this.mbFiring ( true )
	this.fireLaser ( x, y )

--------------------------------------------------------------------------------
end
--------------------------------------------------------------------------------
</pre>

**onStopFiring**

<pre>--------------------------------------------------------------------------------
function laserAI.onStopFiring (  )
--------------------------------------------------------------------------------

	this.mbFiring ( false )

--------------------------------------------------------------------------------
end
--------------------------------------------------------------------------------
</pre>

**onEnterFrame**

This is where it gets fun. Every frame we determine whether the laser should be drawn based on the boolean that is set.  Lasers shouldn't just sit there and look pretty, they should pulse with power!  We make this happen by calculating the amplitude of the wave based on the elapsed time of the game.

<pre>--------------------------------------------------------------------------------
function laserAI.onEnterFrame (  )
--------------------------------------------------------------------------------

	if(this.mbFiring ( ))
	then
		--Cache the object handle
		local hObject = this.getObject ( )

		--Set the laser to be visible
		object.setVisible ( hObject, true )

		--Calculate the width of the laser based on a sine wave
		local amp = math.sin ( application.getTotalFrameTime ( ) * 2000 ) * 0.9 + 2.5

		--Get the current scale of the laser so we can change the width only
		local sx, sy, sz = object.getScale ( hObject )
		object.setScale ( hObject, amp, sy, sz )
	else
		--Set the laser to be invisible since we aren't firing
		local hObject = this.getObject ( )
		object.setVisible ( hObject, false )
	end

--------------------------------------------------------------------------------
end
--------------------------------------------------------------------------------
</pre>

A simple breakdown of the sine wave is:

### math.sin ( application.getTotalFrameTime ( ) * 2000 ) * 0.9 + 2.5

*   math.sin I'm not stupid enough to think I'm going to sum up sine waves in one bullet point.  For the purpose of games, it is helpful to think of the standard arguments of the sine wave using the definition <strong>sine ( w * t)</strong> where:  

> **t** is time

> **w** (omega) is 2 * PI * f

> **f** is the frequency that the wave is oscillating at

*   application.getTotalFrameTime ( ) This provides the total elapsed frame time in seconds since the start of the game.  It is used for time in the equation.
*   **2000 **is the <strong>w </strong>or the <strong>2 * PI * f </strong>component of the wave.  Because I don't want to waste CPU time calculating 2 * PI * f every single frame, I just mooshed these together into a single number.  If you do the math you'll find this frequency is about 318 Hz, not magic by any means, just what I thought looked good in the ShiVa editor.
*  **0.9** This scales the amplitude of the wave.  Essentially it determines how much to exaggerate the pulsation of the laser.  Higher numbers result in more pulsating.
*   <strong>2.5</strong> This sets the fattiness factor of the wave.  Try a value of 10 and you will quickly see!

**fireLaser**

This is where we actually calculate where the laser gets drawn.

<pre>--------------------------------------------------------------------------------
function laserAI.fireLaser ( nPointX, nPointY )
--------------------------------------------------------------------------------

	--See Note 1
	--Find the target z value for the unproject function
	local rx, ry, rz = camera.projectPoint ( application.getCurrentUserActiveCamera ( ), 0, 0, 0 )

	--convert the tapped screen coords to global 3d space
	local x, y, z = camera.unprojectPoint ( application.getCurrentUserActiveCamera ( ), nPointX, nPointY, rz )

	--Get the location of the satellite
	local satx, saty, satz = object.getTranslation ( this.hSat ( ), object.kGlobalSpace )

	--See Note 2
	--Find the distance from the satellite to the point clicked, this is the length of the laser * 2
	--Note my wanky coordinate system, all objects are constrained to Y=0
	local distance = math.vectorLength ( math.vectorSubtract ( satx, 0, satz, x, 0, z ) )

	--Set the translation of the laser to the position of the satellite.  This is easy thanks to the origin
	--point of the laser!
	object.setTranslation ( this.getObject ( ), satx, 0, satz, object.kGlobalSpace )

	--set the scale of the laser to the distance calculated above, being sure to preserve the other scale values
	--that are being modulated in onEnterFrame
	local sx, sy, sz = object.getScale ( this.getObject ( ) )
	object.setScale ( this.getObject ( ), sx, sy, distance/2 )

	--face the beam towards the tap point
	object.lookAt ( this.getObject ( ), x, 0, z, object.kGlobalSpace, 1 )

--------------------------------------------------------------------------------
end
--------------------------------------------------------------------------------
</pre>

**Note 1**: We received x and y coordinates for where the user clicked. These coordinates are in Screen Space which range from -1 to 1. We need to convert these coordinates to global 3d space.  This is called **projection**, specifically we want to project the X, Y mouse coordinates into the 3D space of the current camera.  The one gotcha when doing this is that you have to specify how **deep** within the scene you want to project these points.

The easy way to determine this depth is by picking a known location in 3D space and projecting a point to it.  This will give you the depth of that location in space.  I chose (0,0,0) because depth in my scene is determined only by the Y-axis  my camera is constrained to look directly down on the Y-Axis.  Once  **rz** is obtained, we can turn around and unproject a point using the mouse coordinates + our depth to get the global 3D space coordinates of the tap.

**Note 2:** The length of the beam is simply the distance between the thing shooting the beam and the point where the user tapped.  We can easily find this distance by finding the vector between these 2 points and then getting the length of that vector.  Note that when we set the scale of the laser we need to divide this distance by 2.

## That's It!

If you have done a similar effect, please show me!  If you know of a less-involved way of drawing a laser, please share!
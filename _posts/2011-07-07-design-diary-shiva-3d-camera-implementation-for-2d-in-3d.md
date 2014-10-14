---
title: 'Design Diary: Shiva 3D  Camera Implementation for 2D in 3D'
author: error454
layout: post
permalink: /2011/07/07/design-diary-shiva-3d-camera-implementation-for-2d-in-3d/
categories:
  - Shiva 3D
tags:
  - 2d in 3d
  - angry satellites
  - camera
  - fit objects
  - shake
  - shiva
  - shiva 3d
  - shiva3d
---
This post is part diary and part tutorial.

I will cover:

*   2D in 3D camera basics
*   Camera view extent detection
*   Fitting the camera view to contain certain objects
*   Special FX

<img src='' alt=''>

## 2D in 3D Camera Basics

### Field of View

One of the first things you should consider for your camera that is often overlooked is your field of view (fov). If you take the defaults, you will be set to 35mm. Here's why this is important, the higher your FOV is, the more distorted objects will be that are on the edge of the screen.

See these screenshots for an illustration.

FOV=40 The objects at the edge of the screen are stretched.
<img src='{{ site.url }}/assets/uploads/2011/07/fov40.png' alt=''>

FOV=40 Objects in the center of the screen appear normal.
<img src='{{ site.url }}/assets/uploads/2011/07/fov40centered.png' alt=''>

FOV=22 Objects on the edge of the screen are not stretched as much at lower FOV.
<img src='{{ site.url }}/assets/uploads/2011/07/fov22.png' alt=''>

One reason why you may want to lower your FOV is that it gives an improper sense of perspective in a 2D game. A high FOV may make it appear like an object is underneath another object when it is not. It may also give the user a false sense of distance on the edge of the screen, this is particularly relevant if your playfield wraps (an object goes off the screen to the right and appears on the left side of the screen.

We chose a fov of 22, it had the right amount of depth without making planets look on the periphery look like ovals.

### Up Axis

You need to decide on the orientation of your world. This is personal preference and getting slightly into holy war territory. The common options used are:  
No movement on the Y axis:  
The entire playfield is on the X-Z plane.

No movement on the Z axis:  
The entire playfield is on the X-Y plane.

We chose the first option really for just 1 reason. In Shiva, there is a function** object.lookAt()** which allows you to have an object look at a position in space. For instance, you might have a target defined and you want your drone ship to look at the target, you might do something like this in the drone ship AI:

<pre>--Get the position of the target
local tx, ty, tz = object.getTranslation(this.hTarget(), object.kGlobalSpace)

--Make ourself look at the target
object.lookAt ( this.getObject ( ), tx, ty, tz, object.kGlobalSpace, 1 )
</pre>

The lookAt function orients an object so that the objects -Z axis points at the given coordinate with the +Y axis being up. Take a look at our drone ship model and imagine that the target is in the direction of the squiggly red arrow.

<img src='{{ site.url }}/assets/uploads/2011/07/forward.png' alt=''>

Shiva also provides a lookAtWithUp method where you can define what the Up vector is. I say minimize confusion  after writing a couple thousand lines of code you might rest easier knowing you didn't screw up an Up vector somewhere.

### Camera View Extent Detection

For many 2D games, it is vital to know when an object hits the edge of the screen. For instance, in our game, you can only fire a single rocket at a time. If the player misses a target, the missile flies off into space. We detect when the missile hits the edge of the screen and destroy it so that the player can fire another.

There are many ways to do this. Let me briefly describe the common solutions and then explain which one we chose.

**Render Extents**  
This solution is a simple one that is low precision but may fill certain needs. The idea is to get the bounding sphere representation of each object you are interested in checking and seeing whether that bounding sphere is inside the camera's frustum.

**object.getBoundingSphereRadius(..)** and **object.getBoundingSphereCenter(..)** would be used to get the bounding sphere of the object and the check would be done with **camera.isSphereInFrustum ( hObject, x, y, z, nRadius )**.

The reason I say this is a low precision method is that the check only tells you whether or not the camera can see the object. It doesn't tell you whether the object collided with the right/left-hand side of the screen. Depending on your scene, you could probably calculate this without much trouble.

One issue with this solution is not being able to easily spawn object soutside of the viewable scene. For instance if you wanted to have enemies flying into the scene from outside the viewable area.

**Absolute Position Checking**  
A brute force method is to calculate the translation of the scene extents based on the current position of the camera. Once you know the global space coordinates of the extents, you can loop through objects of interest and see whether they exceed these limits.

This is computationally expensive because you are checking every object every single frame. Also, if your camera is moving, you have to recalculate the global space coordinates every frame as well.

**Extent Sensors**  
We created a custom solution that was more Shiva-centric, in retrospect the solution seems so obvious that I'm sure others have done the same. The basic idea can be summed up by the following photo:

<img src='{{ site.url }}/assets/uploads/2011/07/scene-edges.png' alt=''>

As you can see, we have created a boundary around the edges of the scene. The model that we use for the boundary is nothing more than a 1x1x1 cube with a collision sensor on it, note the origin of the object. Keep this in mind for the following code, we used this origin so that when we scale the object, it grows in one direction instead of growing in two directions (like it would if centered at (0, 0, 0).

<img src='{{ site.url }}/assets/uploads/2011/07/edge-object.png' alt=''>

When the camera is created, we create 4 of these boundaries and position them with the following code:

<pre>--Get a reference point at y = 0, this is where game objects will be
--colliding with the edge of the screen
local rx, ry, rz = camera.projectPoint ( this.getObject ( ), 0, 0, 0 )

--Find the coordinates for the edges of the screen.  This is made easy by
--projecting points from screen space to global space.  Screen space ranges
--from -1 to 1 i.e. the top-right of the screen is (1, 1), bottom-left is (-1,-1)
--top right
local trx, dc, trz = camera.unprojectPoint ( this.getObject ( ), 1, 1, rz )

--top left
local tlx, dc, tlz = camera.unprojectPoint ( this.getObject ( ), -1, 1, rz )

--bottom right
local brx, dc, brz = camera.unprojectPoint ( this.getObject ( ), 1, -1, rz )

--bottom left
local blx, dc, blz = camera.unprojectPoint ( this.getObject ( ), -1, -1, rz )

--Set the translation of our 4 boundary objects so that we can scale them to cover their
--respective edges.
object.setTranslation ( this.mhBoundaryTop ( ), tlx, 0, tlz, object.kGlobalSpace )
object.setTranslation ( this.mhBoundaryBottom ( ), blx, 0, blz, object.kGlobalSpace )
object.setTranslation ( this.mhBoundaryLeft ( ), tlx, 0, tlz + ((blz - tlz)), object.kGlobalSpace )
object.setTranslation ( this.mhBoundaryRight ( ), trx, 0, trz + ((brz - trz)), object.kGlobalSpace )

--Set the scale of the objects so that they cover the width or height of their screen edge
object.setScale ( this.mhBoundaryTop ( ), trx-tlx, 1, 1 )
object.setScale ( this.mhBoundaryBottom ( ), brx-blx, 1, 1 )
object.setScale ( this.mhBoundaryLeft ( ), 1, 1, tlz-blz )
object.setScale ( this.mhBoundaryRight ( ), 1, 1, trz-brz )
</pre>

In the past, I used some pretty ugly trig to calculate the screen edges, using the coordinate transformation is far cleaner. It is important to note that any FOV changes or Y-axis changes will require this to be recalculated. At this point, if you are never going to change the Y position or FOV of your camera, you could parent these objects to the camera and you'd never have to calculate their positions again. To extend this even further, you could set each boundary to have a unique sensor ID and then you'd know exactly which screen edge is being hit.

Finally, be sure to set the visibility of the edge objects to false.

## Fitting The Camera View to Contain Certain Objects

How do you zoom the camera so that it fits a set of objects? It turns out that my solution took me quite a a while to get working due to some of the trig. I am very eager to hear of a simpler model to do this. Here is what I do:

1.  Create an empty table in the camera AI
2.  Send the cameraAI a list of objects to zoom to (they are saved in the table)
3.  Loop over the table, get the bounding box for each object and calculate the minimum and maximum point for all of the objects.
4.  Based on the min and max point, create a helper object that is in the very center of the points.
5.  Calculate the Y value of the helper based on width/height of the screen and the camera FOV.
6.  Move the camera to the helper position

Here is the relevant code:

<pre>local hScene = application.getCurrentUserScene ( )
local nCount = 0
local hCamera = application.getCurrentUserActiveCamera ( )

local fov = camera.getFieldOfView ( hCamera )
local hFov = (2 * math.atan(math.tan(fov) * this.mnScreenWidth ( ) / this.mnScreenHeight ( )))

local width, height = 0, 0
local xMinFinal = 0
local xMaxFinal = 0
local zMaxFinal = 0
local zMinFinal = 0
local yMaxFinal = 0

--Loop over objects
for i = 0, table.getSize ( this.objects ( ) ) - 1
do
	local xmin, ymin, zmin = object.getBoundingBoxMin ( scene.getTaggedObject ( hScene, table.getAt ( this.objects ( ), i )) )
	local xmax, ymax, zmax = object.getBoundingBoxMax ( scene.getTaggedObject ( hScene, table.getAt ( this.objects ( ), i )) )
	width = xmax - xmin
	height = zmax - zmin

	--Prime values if this is the first object
	if( nCount == 0 )
	then
		xMinFinal = xmin
		xMaxFinal = xmax
		zMinFinal = zmin
		zMaxFinal = zmax
		yMaxFinal = ymax
	end

	--Find minimums and maximums
	xMinFinal = math.min ( xMinFinal, xmin )
	zMinFinal = math.min ( zMinFinal, zmin )
	xMaxFinal = math.max ( xMaxFinal, xmax )
	yMaxFinal = math.max ( yMaxFinal, ymax )
	zMaxFinal = math.max ( zMaxFinal, zmax )

	nCount = nCount + 1
end

--Add some padding for the scene extents, simply personal preference
xMinFinal = xMinFinal - 10
xMaxFinal = xMaxFinal + 10
zMinFinal = zMinFinal - 5
zMaxFinal = zMaxFinal + 5

--Set the final distance of the camera
yMaxFinal = yMaxFinal / 2

--Create the helper object if it doesn't exist
if(this.mhZoomHelper ( ) == nil)
then
	this.mhZoomHelper ( scene.createRuntimeObject ( application.getCurrentUserScene ( ), cameraHelper ) )
end

--Calculate camera x, y, z
local x = xMinFinal + (xMaxFinal - xMinFinal) / 2
local z = zMinFinal + (zMaxFinal - zMinFinal) / 2

--Calculate y values for fitting the camera height or width to the scene
local yVert = ((zMaxFinal - zMinFinal) * 0.5) / math.tan ( fov ) + yMaxFinal
local yHoriz = ((xMaxFinal - xMinFinal) * 0.5) / math.tan ( hFov / 2)

--Fit whichever y value is greater - may not be the best choice for all situations
if(yVert  yHoriz)
then
	object.setTranslation ( this.mhZoomHelper ( ), x, yVert, z, object.kGlobalSpace )
else
	object.setTranslation ( this.mhZoomHelper ( ), x, yHoriz, z, object.kGlobalSpace )
end
</pre>

Now you just have to move the camera to the location of this.mhZoomHelper().

## Special FX

Finally, one of the last components of the initial camera implementation was a simple screen shake. For this we simply set a stop time in onEnterFrame and then if the effect is running do something like this:

<pre>local time = application.getTotalFrameTime ( )
local dx = math.sin ( time * 1000 ) * 0.5
local dy = math.sin ( time * 2000 )

object.translate ( this.getObject ( ), dx, dy, 0, object.kLocalSpace )
</pre>

I believe I may have got this basic idea from one of the Shiva included tutorials. Our basic camera implementation has a few more frills that are specific to our game, but I've covered the fundamentals that I would carry over to all future 2D games. You can see our camera shake when destroying drones in this video.
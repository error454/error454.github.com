---
title: Platformer Physics 101 and The 3 Fundamental Equations of Platformers
author: error454
layout: post
permalink: /2013/10/23/platformer-physics-101-and-the-3-fundamental-equations-of-platformers/
categories:
  - Game Theory
  - Shiva 3D
tags:
  - gravity
  - jump
  - jump termination
  - physics
  - platform
  - platformer
  - shiva
---

<script src="http://cdn.stonetrip.com/players/stkobject.js" type="text/javascript"></script>

There are tons of tutorials out there on doing platformer physics and <a href="http://higherorderfun.com/blog/2012/05/20/the-guide-to-implementing-2d-platformers/" target="_blank">implementing various types of platformers</a>.  What there seems to be lacking is a tutorial on how to choose good values for your platformer physics.
<!--more-->
This article will present some core physics equations in a new light!  I will even be so bold as to christen these equations as The Fundamental Equations of Platformers.  The sample code here is presented in LUA and for my prototyping I am using the ShiVa 3D game engine.



When designing physics for a platformer, the 2 fundamental values required are:

1.  The strength of gravity
2.  The initial velocity of a jump

With these values, we're able to use the the kinematic equations to make a character jump and eventually touch the ground again.

## Basic Physics

>**3/22/2014** Please see comment by Ricky below. These simple equations are the calculus versions broken down for simplicity, not the kinematic equations.

As a refresher, the 2 basic equations we'll use are:

<center>
  <img src="http://s0.wp.com/latex.php?latex=velocity%3Dacceleration%5Ccdot+time&bg=ffffff&fg=000&s=2" alt="velocity=acceleration\cdot time " title="velocity=acceleration\cdot time " class="latex" /><br /></br> <img src="http://s0.wp.com/latex.php?latex=distance%3Dvelocity%5Ccdot+time&bg=ffffff&fg=000&s=2" alt="distance=velocity\cdot time" title="distance=velocity\cdot time" class="latex" /> 
</center>This would be a typical implementation.

<pre>-- Set Y velocity to the jump velocity
this.nVelocityY( this.kVelocityJump )</pre>

<pre>-- Apply gravity every frame
local dt = application.getLastFrameTime( )
local newVelocityY = this.nVelocityY( ) - this.kGravity( ) * dt
local distanceToMoveY = newVelocityY * dt

-- Assuming collision detection was ok, move the actor
object.translate( this.getObject( ), 0, distanceToMoveY, 0, object.kGlobalSpace )
this.nVelocityY( newVelocityY )</pre>

This code is at the heart of platformer physics, but when it comes down to it, these equations alone kind of suck at being useful. Picking random values isn't the best way to get a good feeling platformer. Let's look at a method that will help get you started with initial physics values.

## The Fundamental Equations of Platformers

What makes more sense is to calculate gravity and initial jump velocity by picking 2 simple properties of the universe:

1.  Max jump height
2.  Time to reach max height

Which leads us to the first two fundamental equations:

<center>
  <img src="http://s0.wp.com/latex.php?latex=gravity%3D%5Cdfrac%7B2%5Ccdot+height_%7BMaxJump%7D%7D%7Btime_%7BToApex%7D%5E2%7D%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%281%29&bg=ffffff&fg=000&s=2" alt="gravity=\dfrac{2\cdot height_{MaxJump}}{time_{ToApex}^2}\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;(1)" title="gravity=\dfrac{2\cdot height_{MaxJump}}{time_{ToApex}^2}\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;(1)" class="latex" /> 
</center>

<center>
  <img src="http://s0.wp.com/latex.php?latex=V_%7Bjump%7D%3D%5Csqrt%7B2%5Ccdot+gravity%5Ccdot+height_%7BMaxJump%7D%7D%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%282%29&bg=ffffff&fg=000&s=2" alt="V_{jump}=\sqrt{2\cdot gravity\cdot height_{MaxJump}}\;\;\;\;\;\;(2)" title="V_{jump}=\sqrt{2\cdot gravity\cdot height_{MaxJump}}\;\;\;\;\;\;(2)" class="latex" />
</center>So what we've defined above are:

*   gravity as a function of the time to reach the top of the jump and maximum jump height.
*   initial jump velocity as a function of gravity and maximum jump height

###The Derivations
**Equation 1**
Assuming that we are standing on the ground preparing to jump, we start with the kinematic equation <img src="http://s0.wp.com/latex.php?latex=d%3Dv_it+%2B+%5Cdfrac%7B1%7D%7B2%7Dat%5E2+&bg=ffffff&fg=000&s=2" alt="d=v_it + \dfrac{1}{2}at^2 " title="d=v_it + \dfrac{1}{2}at^2 " class="latex" />

Setting initial velocity to zero <img src="http://s0.wp.com/latex.php?latex=d%3D%5Cdfrac%7B1%7D%7B2%7Dat%5E2+&bg=ffffff&fg=000&s=2" alt="d=\dfrac{1}{2}at^2 " title="d=\dfrac{1}{2}at^2 " class="latex" />

Solving for **a** yields
<img src="http://s0.wp.com/latex.php?latex=a%3D%5Cdfrac%7B2d%7D%7Bt%5E2%7D+&bg=ffffff&fg=000&s=2" alt="a=\dfrac{2d}{t^2} " title="a=\dfrac{2d}{t^2} " class="latex" />

**Equation 2**
Assuming that we are standing on the ground preparing to jump, we start with the kinematic equation
<img src="http://s0.wp.com/latex.php?latex=v_f%5E2%3Dv_i%5E2%2B2ad+&bg=ffffff&fg=000&s=2" alt="v_f^2=v_i^2+2ad " title="v_f^2=v_i^2+2ad " class="latex" />

Setting initial velocity to zero
<img src="http://s0.wp.com/latex.php?latex=v_f%5E2%3D2ad+&bg=ffffff&fg=000&s=2" alt="v_f^2=2ad " title="v_f^2=2ad " class="latex" />

Solving for **v** yields
<img src="http://s0.wp.com/latex.php?latex=v_f%3D%5Csqrt%7B2ad%7D+&bg=ffffff&fg=000&s=2" alt="v_f=\sqrt{2ad} " title="v_f=\sqrt{2ad} " class="latex" />

##Early Jump Termination
**Edited 3/5/2014**

As reader Chue points out, the equations below don't work unless **gravity is negative**. This is true and was an oversight on my part. You could redefine the above equation for gravity and just tack a negative sign on it. This also means that the equation for jump velocity is going to yield an imaginary number (square root of a negative number), you can just throw that imaginary part away. I've updated the equations below to show that gravity is being plugged in as a negative number.

The end numbers haven't changed, I just failed to show my work. The spreadsheet at the end of the article was not affected.
        
There are a few ways to do early jump termination but I am going to propose a method that is based on a single parameter, **minimum desired jump height**.  The idea is to calculate the downward velocity required to achieve this minimum height. I now present the 3rd fundamental platformer equation (which is a well known kinematic equation):
          <center>
            <img src="http://s0.wp.com/latex.php?latex=V_%7BEarlyJumpTermination%7D%3D%5Csqrt%7BV_%7Bjump%7D%5E2+%2B+2%5Ccdot+gravity+%28height_%7BMaxJump%7D+-+height_%7BMinJump%7D%29%7D%5C%3B%5C%3B%283%29&bg=ffffff&fg=000&s=2" alt="V_{EarlyJumpTermination}=\sqrt{V_{jump}^2 + 2\cdot gravity (height_{MaxJump} - height_{MinJump})}\;\;(3)" title="V_{EarlyJumpTermination}=\sqrt{V_{jump}^2 + 2\cdot gravity (height_{MaxJump} - height_{MinJump})}\;\;(3)" class="latex" />
          </center>
          
To use this, we choose our minimum jump height of 1 unit and we arrive at:
          <center>
            <img src="http://s0.wp.com/latex.php?latex=V_%7BEarlyJumpTermination%7D%3D%5Csqrt%7B16%5E2+%2B+2%5Ccdot+-32+%284+-+1%29%7D+%3D+8&bg=ffffff&fg=000&s=2" alt="V_{EarlyJumpTermination}=\sqrt{16^2 + 2\cdot -32 (4 - 1)} = 8" title="V_{EarlyJumpTermination}=\sqrt{16^2 + 2\cdot -32 (4 - 1)} = 8" class="latex" />
          </center>
          
So then in code, when a player releases the jump key, you'd do the following:
        
<pre>if( this.nVelocityY ( ) > 0 ) then
    -- Set velocity to whatever is smaller, termination velocity or current velocity
    this.nVelocityY ( math.min ( this.kJumpVelocityTermination ( ), this.nVelocityY ( ) ) )
end</pre>
        
The one caveat with this method is that, depending on your world, you may set a minimum jump height that is impossible for a human to hit. Because obviously we have limitations, like not being able to press and release a key much faster than 200 milliseconds. So, as a friendly check, we can calculate the last possible second that a jump can be terminated using this equation:

<img src="http://s0.wp.com/latex.php?latex=t%3Dtime_%7BToApex%7D+-+%5Cdfrac%7B2%28height_%7BMaxJump%7D+-+height_%7BMinJump%7D%29%7D%7BV_%7Bjump%7D%2BV_%7BEarlyJumpTermination%7D%7D&bg=ffffff&fg=000&s=2" alt="t=time_{ToApex} - \dfrac{2(height_{MaxJump} - height_{MinJump})}{V_{jump}+V_{EarlyJumpTermination}}" title="t=time_{ToApex} - \dfrac{2(height_{MaxJump} - height_{MinJump})}{V_{jump}+V_{EarlyJumpTermination}}" class="latex" />
          

##Demo
To prove out the equations and method, I did my best to recreate the physics from Mario 1 inside the ShiVa 3D game engine.  Based on my reverse engineering efforts (see details below) I arrived at the following values:
* Max Jump Height = 4 units
* Time to reach max height = 0.44 seconds
* Minimum jump height = 1 unit

So plugging these into equations 1, 2 and 3 we get
<img src="http://s0.wp.com/latex.php?latex=gravity%3D%5Cdfrac%7B2%5Ccdot+4%7D%7B0.44%5E2%7D+%3D+41.322&bg=ffffff&fg=000&s=2" alt="gravity=\dfrac{2\cdot 4}{0.44^2} = 41.322" title="gravity=\dfrac{2\cdot 4}{0.44^2} = 41.322" class="latex" />
          
<img src="http://s0.wp.com/latex.php?latex=V_%7Bjump%7D%3D%5Csqrt%7B2%5Ccdot+41.322%5Ccdot+4%7D%3D18.182&bg=ffffff&fg=000&s=2" alt="V_{jump}=\sqrt{2\cdot 41.322\cdot 4}=18.182" title="V_{jump}=\sqrt{2\cdot 41.322\cdot 4}=18.182" class="latex" />
          
<img src="http://s0.wp.com/latex.php?latex=V_%7BEarlyJumpTermination%7D%3D%5Csqrt%7B18.182%5E2+%2B+2%5Ccdot+-41.322+%284+-+1%29%7D%3D9.091&bg=ffffff&fg=000&s=2" alt="V_{EarlyJumpTermination}=\sqrt{18.182^2 + 2\cdot -41.322 (4 - 1)}=9.091" title="V_{EarlyJumpTermination}=\sqrt{18.182^2 + 2\cdot -41.322 (4 - 1)}=9.091" class="latex" />
          
Now see it in action, spacebar jumps.
        
<script  language="javascript" type="text/javascript">
   stkobject( "800" , "600" , "{{ site.url }}/assets/shiva/PlatformerPhysicsTuned.stk" , null, null , null , null , null , null , 0 , 1 , "<V t='2' n='S3DStartUpOptions.BackgroundColor'>034,034,034</V>" , 0 , 0 , 0 , 0 , 1, null , null , ".png", 0 , 222222,1);
</script>
        
##How I measured Mario Gravity
Starting with Nestopia, I recorded a video of mario jumping.  I then saved that out as an AVI with uncompressed frames.  After pulling that into VLC Media Player, I used the scene filter to save out every frame of the video to a png.

The video was recorded at 50 frames/second which was a bit overkill, but this gave me 0.02 sec/frame.

Instead of counting everything in pixels or trying to convert to meters.  I decided to use units of game height.  In Mario, every object is based on a square of 16x16 pixels.  So I called this a game unit. Using photoshop, I overlaid a grid to display individual pixels. Notice that mario sits 1 pixel into the ground.

<a href="{{ site.url }}/assets/uploads/2013/10/mario0.jpg"><img alt="Mario Units" src="{{ site.url }}/assets/uploads/2013/10/mario0-200x300.jpg"></a>
          
I then proceeded to make some basic measurements.  The time to reach the apex of a jump occurred in 22 frames * (0.02 sec/frame) = 0.44 sec

The total height traveled in a jump was 64 px / 16 = 4 units

It was a head scratcher to first determine how to measure the jump distance of mario based on how his animation changes when he jumps.  I decided to measure using the bottom of mario's feet when standing and the bottom of his lower foot when jumping.  This is because when standing, mario's feet extend 1 pixel into the terrain and while jumping, his bottom foot has a 1 pixel toe that at the highest jump point will extend 1 pixel into any reachable obstacle.

So then the gravity for mario is 2h / t^2 = 41.32 units/s^2

And then initial jump velocity is sqrt(2gh) == sqrt(2*41.32*4) == 18.182

I thought I'd look up the values for Mario's gravity and see if perhaps I could compare my measurements.  I was excited when I found <a href="http://hypertextbook.com/facts/2007/mariogravity.shtml" target="_blank">Acceleration Due to Gravity: Super Mario Brothers</a> until I saw their measurement of the height of mario to be 39 pixels.  Perhaps this could have been due to over or underscan on their TV?  I mean, this value isn't even a power of 2.  Mario is 1616 small and 1632 with a mushroom, everyone knows that!

##Conclusion
This method may not match the exact jump model that mario uses, but as an approximation, I think it works very well. I've created a <a href="https://docs.google.com/spreadsheet/ccc?key=0AoGqxtUhFBJDdENYbWUyd1NZa3dKRThnTHlyZHVLMnc&usp=sharing" target="_blank">Google Docs template</a> with the above equations built-in. Feel free to make a copy of it to calculate some starting values for your own platformer. Also if you've used the above method or see an issue with implications of the early jump termination method, please leave a comment!
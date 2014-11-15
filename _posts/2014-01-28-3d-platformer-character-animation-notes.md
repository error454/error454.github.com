---
title: 3D Platformer Character Animation Notes
author: error454
layout: post
permalink: /2014/01/28/3d-platformer-character-animation-notes/
categories:
  - Game Theory
  - Graphics
  - Shiva 3D
tags:
  - 3d
  - aiming
  - animation
  - platformer
  - shiva
---

<script src="http://cdn.stonetrip.com/players/stkobject.js" type="text/javascript"></script>

<a href='{{ site.url }}/assets/uploads/2014/01/Capture24.jpg'><img src='{{ site.url }}/assets/uploads/2014/01/Capture24-1024x576.jpg' alt='A picture of some cool cartoony 3D models (they look like the alien from bugs bunny) and they&apos;re shooting a totally ridiculous number of bullets all over the screen. I mean the number of bullets is TOTALLY ridiculously awesome!'></a>

We made a 3D platformer with a constrained 2D view not long ago, obviously in our engine of choice, <a href="http://www.shivaengine.com" target="_blank">ShiVa</a>.  The main character fired a number of different ranged weapons and generally made a huge mess of some unfriendly boxes that were invading his planet.
<!--more-->
I faced a number of challenges when implementing the character running, aiming and shooting animations and I thought it would be useful to share my solutions for the next indie.  In this post is a web demo, you'll need to let your browser load the shiva plugin to see it.

The primary topics that I'll discuss are:

<ul type="disc">
  <li>
    Syncing character run speed and movement rate
  </li>
  <li>
    Aiming the gun
  </li>
  <li>
    Shooting the gun
  </li>
  <li>
    Animation track mixing
  </li>
</ul>



# Run Speed & Movement Rate

I remember when the first Unreal Engine came out, there was an article with one of the ID dudes (I want to say it was Carmack) and he was kind of slamming Unreal Engine because their monsters slid across the ground.  Basically, the rate that the character was being translated across the ground and the playback rate of the walk cycle were not synced up to look good.  I like to think of this relationship between animation speed and movement rate as a zone that will from now on be called the Zone of Awesomeness.

The Zone of Awesomeness (ZoA) is where you want to be, it's the fine green line running down the center of the graph.  If the animation speed becomes faster than the movement rate and goes above the line, the character begins to look like s/he's running on ice.  If the animation speed drops below the movement rate, the character looks like s/he is sliding.  There are times when you may want to have the running on ice effect, (you know like when you're actually running on ice) but you'll rarely ever want the sliding effect (unless your game is about ice skating).

<img src='{{ site.url }}/assets/uploads/2014/01/theAzone.jpg' alt='Animation Speed &gt; Movement Rate = Ice Animation Speed &lt; Movement Rate = Sliding'>

## Staying in the ZoA

Our game had a speed boost which increases the movement rate of the character.  I wanted the animation to automatically adapt to the character's velocity so that we stayed in the ZoA.  Below is the method that I used to do this along with the prototype I made to plug values in.

Like most prototypes, the concept was simple enough, it required:

1.  A scene where the character can run left/right and allow me to clearly see his feet
2.  Ability to modify character movement rate
3.  Ability to modify animation speed

**Controls**

*   1/2: Decrease/Increase Movement Rate
*   3/4: Decrease/Increase Animation Speed
*   A/D: Move left/right
*   Mouse: Aim/fire
*   9: Toggle auto-animation speed calculation

<script  language="javascript" type="text/javascript">
   stkobject( "800" , "600" , "{{ site.url }}/assets/shiva/charanim.stk" , null, null , null , null , null , null , 0 , 1 , "<V t='2' n='S3DStartUpOptions.BackgroundColor'>034,034,034</V>" , 0 , 0 , 0 , 0 , 1, null , null , ".png", 0 , 222222,1);
</script>

## Finding an Equation for the ZoA

The goal then is to have a function where you pass in movement velocity and you get back the correct animation speed:

Animation Speed = f(Velocity)

To arrive at this equation for animation speed, I could have done a bunch of math, but I took the lab approach. I decided to collect data and then interpret the results.  Here was my Character Animation Lab 101:

<ol type="1">
  <li value="1">
    Pick a velocity for the player
  </li>
  <li>
    Adjust animation speed until it looks good
  </li>
  <li>
    Log the player velocity and animation speed
  </li>
  <li>
    Perform steps 1-3 for several different velocities that will be used in the game
  </li>
</ol>

This gave me the following set of data values.  As you can see, I charted the values and then using the cheating powers of excel, I added a trendline to the chart and selected linear regression.   It was pretty clear that this data was linear and not a polynomial, exponential or quadratic!  Notice that my R² value is 0.9997 which indicates that this is a good fit.

<img src='{{ site.url }}/assets/uploads/2014/01/regression.jpg' alt='We find the ZoA equation using linear regression'>

If my R² value was < 0.9 then I may have used a series of linear equations based on the velocity range that I was in.  Perhaps switching between 2 or 3 equations based on velocity ranges.  It really depends on what the line looks like.

To wrap up, my equation for animation speed was:

animation speed = 2.9 * velocity * 0.56

To see it in action in the demo, press the 9 key to turn on the animation calculation and then change the run velocity, magic!

# Aiming the Gun

<img src='{{ site.url }}/assets/uploads/2014/01/Sketch25625851.png' alt='Character reference angles. Conceptually the math was easier with 0 degrees being the start of the animation and 180 being the end.'>

Aiming the gun was one of the more conceptually difficult problems to solve. How do we animate the character across a full 180 degree arc (facing one direction) so that he aims exactly where the player is pointing? Now how do we do this while he's running/jumping?

We considered manually transforming bones in code to move arms, gun and head where the mouse or analog stick was aimed.  The problem with this method is obviously that a ton of bones move when you're aiming, pretty much the entire upper body.  So things looked pretty robotic doing this and it turns out that my code animation skills are not the best.

I scoured the internet for ideas on how to approach what would seem like a common problem aiming the gun of a character but surprisingly came up empty. I studied the rogue in Trine 2 for quite some time, being about as close as I could get to a well implemented reference. After a lot of scribbling and head scratching, I came up with my own solution.  To the reader that's just now typing in the comments Why didn't you use XYZ Gamasutra article from 1992? /shakesfist where were you 6 months ago :(

I told my brother to make an animation 360 frames long where the character start position was aiming straight up and the ending position was the character aiming straight down at his feet.  This animation only involved the upper body, so I deleted the un-used animation tracks so as not to interfere with say running.  Also it was important that the animation moved at a constant speed with no tweening at the start or end.

This animation gave me a full 180 degree range of motion, every possible angle I needed to aim! I wanted it 360 frames long so that I would have double resolution for every degree of animation, although in retrospect this may have been overkill. Now, instead of triggering this animation and letting it play out, I manually controlled the exact keyframe it was on based on the angle that the character was aiming.

In practice, the whole thing was one simple interpolation, with the most difficult part being converting the incoming angles so that the origin was 0 degrees when aiming straight up. Doing this made the remaining math much easier because then I could easily interpolate between the minimum and maximum keyframes by the number of degrees / 180:

<pre>local nCurrentKeyFrame = math.interpolate ( 0, 360, nDegrees / 180 )</pre>

# Shooting the Gun

I was hoping that our gun positioning would be so precise that all I had to do was spawn a projectile at the tip of the gun and let it rip across the screen. Maybe this is just noob wishful thinking because this wasn't the case, although I tried it at first. What ended up happening was that projectiles couldn't hit anything, they were off by just a fraction of a degree and would go too far into the screen depth wise.  Also, when running, the gun bobs and pitches ever so slightly and this makes using the gun as a reference point inaccurate.

I settled on a method that I'm calling the virtual aiming cursor. The goal is to position a helper object where the projectile will be spawned and have it look at the destination. Doing this makes it easy to use a mouse or analog stick to do the aiming, much of the logic is the same.

For the mouse, I find the vector from the camera to the mouse cursor.  I then do a ray-plane intersection to find the actual point on the 2D plane where my character should be aiming, you have to do this to make sure your projectiles stay in the correct plane of existance!  I then used the head bone on the character and find the vector from the head to the intersection point on the 2D plane.  Once I have this, I calculate a vector offset to compensate for the length of the gun, so instead of spawning the bullet on my character's head, it spawns near the tip of the gun. From here, position the helper, have it look at the intersection point and we're ready to fire!

This method was close enough and most importantly prioritizes accuracy.  The projectile may not be perfectly matched to the barrel, but it always goes exactly where you are pointing.

# Animation Tracks

The final piece of the puzzle was mixing animation tracks.  Or more accurately, deleting animation tracks.  I had to hand delete a number of conflicting animation tracks.  For instance in the run animation, I deleted all the tracks that the aim animation uses.  In the aiming animation, I deleted all of the lower body tracks so that the run animation could take over.

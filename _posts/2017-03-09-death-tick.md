---
title: The Death of Tick&#58; UE4 & The Future of Programming
author: error454
layout: post
published: true
permalink: /2017/03/09/death/tick
categories:
  - ue4
  - c++
  - sk
tags:
  - ue4
  - c++
  - sk
  - skookumscript
  - scripting language
  - blueprint
  - compile times
  - rapid prototyping
---
Last year I was so frustrated with compile times in UE4 that I hacked together [a script](https://gist.github.com/error454/1eed2db4cce45716c8c5#file-unrealbuildhue-py-L40) that changed the color of one of the lights in my office to let me know when a compile had finished. I also stashed away the compile times in a log file so I could analyze them later.

In the ~6 month period from June 26, 2015 - January 16, 2016, I spent 44.98 hours compiling across a total of 4,660 compiles. This averages to 34.75 seconds/compile but it doesn't account for the time taken to reload UE4 and get back to what I was working on when a hot-reload failed (~25% chance) or when I forgot to check my pointers. This utter frustration with C++ iteration time in UE4 led me to discover a somewhat ridiculously named programming language called SkookumScript.

All I wanted was to reduce my C++ iteration times with a simple scripting language that I could use in UE4. Instead, I underwent a complete programming paradigm shift. And when you find something this profound and amazing, you just have to share it with others. 
<!--more-->
This is a big topic and I can't cover everything in a single article, so this article will cover what I consider to be the 4 pillars of the language. I hope to make this fun and interesting and so am relying more on short code snippets with accompanying short videos to illustrate these 4 pillars.

So, if you use Unreal Engine 4 (or a custom C++ engine), grab a hot drink and prepare yourself. You may shortly feel the fabric of your reality being slowly torn apart along with the urge to resist change, this is a normal part of paradigm shifts, it's just your primitive ego trying to protect itself. Sit on it, contemplate, take time to process, cry if you need to and return when you're prepared.

# Tick

Before we talk about SkookumScript, we need to discuss our friend, the tick. As game developers, we live and die by the tick. Tick is an obvious necessity for everything we do in our game but have you ever stopped to consider how terrible tick is? Jean Simonet masterfully points out in his article [Logic Over Time](http://www.gamasutra.com/blogs/JeanSimonet/20160128/264083/Logic_Over_Time.php), how tick forces us to break up our logic so that it can persist across ticks. This is ultimately a very unnatural way to write logic. I want to look at a simple algorithm to illustrate how tick governs its implementation.

Imagine a homing missile that uses the following algorithm when fired, here separated into 3 stages of existence.

**Stage 1**

* Apply an impulse in the forward direction
* Wait for half a second

**Stage 2**

* Acquire a target
* Home to the target

**Stage 3**

* Destroy self after 10 seconds if no target has been hit

Because of tick, your pseudo code might look something like this:

```
Begin()
{
  Stage = 1
  FireImpulse()
  SetTimer Stage1Over for 0.5 seconds
  SetTimer Stage2Over for 10 seconds
}

Tick()
{
  if Stage == 2
  {
    HomeToTarget()
  }
}

Stage1Over()
{
  Stage = 2
  AcquireTarget()
}

Stage2Over()
{
  DestroySelf()
}
```

Now, by looking at the code alone, try to piece together what the original algorithm was supposed to be. Gleaning the intent of an algorithm can become tedious as complexity increases. We have patterns that help better structure this complexity and these state transitions, but at the end of the day, tick forces us to take our easily understood algorithms and splatter them across our codebase like some expressionist artist flinging paint onto a canvas.

<iframe src="//giphy.com/embed/CTVr5GltEmS76" width="480" height="383" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

There's a better way, an alternative that will change your entire perspective of writing game logic. Here's the actual SkookumScript (Sk) code for this algorithm:

```javascript
()
[
  fire_forward_impulse
  _wait(0.5)
  acquire_target

  race
  [
    _home_to_target
    _wait(9.5) // we already waited 0.5 seconds above
  ]

  destroy_self
]
```

This reads like plain English, like the original algorithm. The inquisitive reader is now wondering, what do the underscores mean? What does the ```race``` keyword do? Let's dive in, but first, here are some official links for Sk:

* [What is Sk?](http://skookumscript.com/about/)
* [What are the primary features?](http://skookumscript.com/about/features/)
* [How did it come about?](http://skookumscript.com/about/origin/)
* [Who are the creators? Did they actually know what they were doing?](http://skookumscript.com/about/team/)

Sk is an awesome programming language, it's easy to code in, it's got concurrency features built in, it's timesliced not multithreaded so you don't worry about mutexes and locking, it can call your C++, it can call blueprints, blueprints can call it, it's fast, etc etc. Go read their web page to learn more. What I want to focus on for the remainder of this article is actually *showing* you how this stuff works by using small code snippets + videos from my game.

# Firing Loop Examples

The following examples will involve modifying the ```_fire``` coroutine. This coroutine gets called in an endless loop like so:

```
loop
[
  if @is_firing? // @ indicates a member variable while ? indicates a boolean
  [
    _fire
  ]
  else [_wait] // wait 1 frame
]
```

## The Basics

```
fire_projectile(1.0) // 1.0 is the size of the projectile
_wait(0.15)
```

<iframe width="512" height="512" src="https://www.youtube.com/embed/VtkJ9lpA7QI?rel=0&amp;controls=0&amp;showinfo=0&amp;autoplay=1&amp;loop=1&amp;playlist=VtkJ9lpA7QI" frameborder="0" allowfullscreen></iframe>

### Immediate vs Deferred

Sk has the concept of immediate statements and deferred statements. An immediate statement completes in the same frame (like every function you've ever used) while a deferred statement can take more than a single frame to complete. Deferred statements are called coroutines and begin with an underscore. ```_wait``` is the built in coroutine that waits for a specified amount of time:

```
_wait // wait 1 frame
_wait(1) // wait 1 second
```

Note that parentheses are not required if a method or coroutine takes no arguments (or if default arguments are desired). In the basic example above, you see both an immediate statement ```fire_projectile(1.0)``` and a deferred one ```_wait(0.15)```.

If you tried to call ```_wait``` inside the method ```fire_projectile```, you'd get a compile error because a deferred statement cannot be called from within an immediate. However, you could just as easily create a ```_fire_projectile``` coroutine where waiting is allowed. Alternatively, you could ```branch``` off a new coroutine from within the immediate (more on that later).

## 3-round Burst with Reload Delay

```
// Do something 3 times with the ability to wait
// There's also an immediate version .do
3._do
[
  fire_projectile(1.0)
  _wait(0.05)
]
_wait(0.5)
```

<iframe width="512" height="512" src="https://www.youtube.com/embed/_mBtggWfdcc?rel=0&amp;controls=0&amp;showinfo=0&amp;autoplay=1&amp;loop=1&amp;playlist=_mBtggWfdcc" frameborder="0" allowfullscreen></iframe>

### do

The do keyword literally means, do something X times.

```
5.do[println("HI")] // prints HI 5 times
3.do[println(idx)] // prints 0 1 2

// print HI 3 times with a 1-second delay between each
3._do
[
  println("HI") 
  _wait(1)
]
```

You can also use a variable to specify the number of iterations. Note that ```idx``` is a built-in variable that provides the current index in the ```do```. [It's just a closure](https://github.com/error454/SkookumScript-UnrealEngine/blob/master/Scripts/Core/Object/Integer/do().sk) if you are curious.

```
variable.do[println(idx)] // prints 0 1 2 ... variable - 1
```

## Spread Shot

```
// Get the rotation of our gun socket:
//  ! is used to construct a new object
//  : binds a variable to an object
//  @ is used to access a member variable of an object, @rob_skeleton is
//   a skeletal mesh component that is part of our Pawn class.
//  "LeftGunSocket".Name converts the string to an FName which the built-in
//   socket_rotation method expects.
!rot : @rob_skeleton.socket_rotation("LeftGunSocket".Name)

// pass in the rotation and scale of the projectile
// rot.@yaw accesses the yaw variable of the RotationAngles object
fire_projectile_degrees(rot.@yaw, 1.0)
fire_projectile_degrees(rot.@yaw + 5.0, 1.0)
fire_projectile_degrees(rot.@yaw - 5.0, 1.0)
_wait(0.5)
```

<iframe width="512" height="512" src="https://www.youtube.com/embed/__JnxJcmBic?rel=0&amp;controls=0&amp;showinfo=0&amp;autoplay=1&amp;loop=1&amp;playlist=__JnxJcmBic" frameborder="0" allowfullscreen></iframe>

## Burst + Grand Finale

```
3._do
[
  fire_projectile(1.0)
  _wait(0.05)
]

_wait(0.3)
100.do
[
  // idx>> converts idx (Integer) to a Real, I could write idx>>Real but Sk is
  // smart enough to convert to the correct type based on the context.
  fire_projectile_degrees([360.0 / 100.0] * idx>>, 1.0)
]
_wait(1.0)
```

<iframe width="512" height="512" src="https://www.youtube.com/embed/E3L7NFp6uaQ?rel=0&amp;controls=0&amp;showinfo=0&amp;autoplay=1&amp;loop=1&amp;playlist=E3L7NFp6uaQ" frameborder="0" allowfullscreen></iframe>

## Burst + Circle + Finale

```
3._do
[
  fire_projectile(1.0)
  _wait(0.05)
]

50._do
[
  fire_projectile_degrees([360.0 / 50.0] * idx>>, 0.8, blue)
  _wait(0.05)
]

_wait(0.3)
100.do
[
  fire_projectile_degrees([360.0 / 100.0] * idx>>, 1.0)
]
_wait(1.0)
```

<iframe width="512" height="512" src="https://www.youtube.com/embed/R3cDncgacmI?rel=0&amp;controls=0&amp;showinfo=0&amp;autoplay=1&amp;loop=1&amp;playlist=R3cDncgacmI" frameborder="0" allowfullscreen></iframe>

## Burst + Sync[Circle Circle] + Finale

This one has something new, see if you spot it.

```
3._do
[
  fire_projectile(1.0)
  _wait(0.05)
]

sync
[  
  50._do
  [
    fire_projectile_degrees([360.0 / 50.0] * idx>>, 0.8, blue)
    _wait(0.05)
  ]

  75._do_reverse
  [
    fire_projectile_degrees([360.0 / 75.0] * idx>>, 0.8, green)
    _wait(0.05)
  ]
]

_wait(0.3)
100.do
[
  fire_projectile_degrees([360.0 / 100.0] * idx>>, 1.0)
]
_wait(1.0)
```

<iframe width="512" height="512" src="https://www.youtube.com/embed/Sq320qHfuQU?rel=0&amp;controls=0&amp;showinfo=0&amp;autoplay=1&amp;loop=1&amp;playlist=Sq320qHfuQU" frameborder="0" allowfullscreen></iframe>

The above code block introduces a feature of Sk called ```sync```. ```sync``` lets you specify a block of coroutines to fire off in parallel. Execution only continues beyond the ```sync``` block once *all* of the coroutines in the block have completed.

```
sync
[
  _routine1
  _routine2
  _routine3
]
// We don't reach this line until routine1-3 have completed.
```

In my case, I'm using ```_do``` to define my 2 coroutines. I might refactor this once I nailed down the patterns I like, so I might make a ```_circular_pattern``` coroutine that takes the quantity, color, delay and scale:

```
// Production code would look more like this
_burst_shot

sync
[
  _circular_pattern(50, blue, 0.05, 0.8)
  _circular_pattern(75, green, 0.05, 0.8)
]

_grand_finale
```

## Burst + Race[Circle Circle Wait] + Finale

```
3._do
[
  fire_projectile(1.0)
  _wait(0.05)
]

race
[  
  50._do
  [
    fire_projectile_degrees([360.0 / 50.0] * idx>>, 0.8, blue)
    _wait(0.05)
  ]
  
  75._do_reverse
  [
    fire_projectile_degrees([360.0 / 75.0] * idx>>, 0.8, green)
    _wait(0.05)
  ]
  _wait(1)
]
  
_wait(0.3)
100.do
[
  fire_projectile_degrees([360.0 / 100.0] * idx>>, 1.0)
]
_wait(1.0)
```

<iframe width="512" height="512" src="https://www.youtube.com/embed/deOPD4l3J5w?rel=0&amp;controls=0&amp;showinfo=0&amp;autoplay=1&amp;loop=1&amp;playlist=deOPD4l3J5w" frameborder="0" allowfullscreen></iframe>

### race

```race``` lets you specify a block of coroutines to fire off in parallel. When *any* of these coroutines exit, all other coroutines in the ```race``` block are terminated and execution continues past the ```race``` block. It's literally like a race where all coroutines are competing to be the first one to exit. As an example, I've taken the same code as the ```sync``` example and ```race```'d it against a ```_wait(1)```. The expectation would be that ```_wait(1)``` finishes before the circular firing patterns and therefore the entire race block should exit after 1 second.

Here's another use for ```race```, try to guess what it does.

```
race
[
  _wait_not_firing
  _fire
]
```

Did you get it? It aborts the entire ```_fire``` coroutine once the user lets off the fire button.

Where ```_wait_not_firing``` might look like this:

```
() 
[
  loop
  [
    [exit] when not @is_firing?
    _wait
  ]
]
```

## Branch

This final example wraps up with an illustration of using ```branch```. ```branch``` fires off a coroutine in the background and immediately continues to the next statement. Here I'm using it to attach some additional behavior to the grand finale projectiles. I've also changed up the firing pattern in the ```sync``` statement and added an example of overriding the default closure variable name to allow for a nested ```_do``` (you can investigate on your own). Use the colors as your guide to see what each statement is doing.

```
3._do
[
  fire_projectile(1.0)
  _wait(0.05)
]
  
sync
[  
  50._do
  [
    fire_projectile_degrees([360.0 / 50.0] * idx>>, 0.8, blue)
    _wait(0.05)
  ]
 
  250._do_reverse
  [
    fire_projectile_degrees([[5.0 * 360.0] / 250.0] * idx>>, 0.8, green)
    _wait(0.01)
  ]
   
  4._do
  [
    5.do (Integer idx2)
    [
      fire_projectile_degrees([[90.0 / 5.0] * idx2>>] + [90.0 * idx>>], 2.0, purple)        
    ]
    _wait(0.8)
  ]
]
  
_wait(0.3)
100.do
[
  !p : fire_projectile_degrees([360.0 / 100.0] * idx>>, 1.0, purple)
  branch
  [
    _wait(0.2)
    if p.not_nil?
    [
      p._seek_actor(this) // this is the pawn that is firing these projectiles
    ]      
  ]
]
_wait(1.0)
```

<iframe width="512" height="512" src="https://www.youtube.com/embed/Qt4HQd-bg-k?rel=0&amp;controls=0&amp;showinfo=0&amp;autoplay=1&amp;loop=1&amp;playlist=Qt4HQd-bg-k" frameborder="0" allowfullscreen></iframe>

## Wrapup

That was a crash course in what I consider the pillars of Sk: ```race```, ```sync```, ```branch``` and ```_wait```. But I've only covered a small fraction of the features that are crammed into the language. Hopefully, you've seen how removing Tick can allow for writing very expressive and simple algorithms.

I'm sure you might have questions or even criticisms. I've included some popular ones below with a link to their official answers. If you have further questions on the language, particularly deep questions, I'd recommend signing up on the forums and asking there, where you'll get a response directly from the mad scientists themselves.

* [What about performance?](http://forum.skookumscript.com/t/skookumscript-performance/500/5?u=error454)
* [Is it compiled or interpreted?](http://skookumscript.com/about/faq/#qst-compiled)
* [Is it statically or dynamically typed?](http://skookumscript.com/docs/v3.0/lang/typesystem/)
* [How do you control groups of coroutines?](http://forum.skookumscript.com/t/the-mind-the-master-and-who-pulls-the-strings-in-skookumscript/110)

I didn't even get around to showing how amazingly fast Sk is to iterate on or how useful it is as a super powered command prompt. All of these firing patterns were made in a live running game, where hot-recompiling literally took less than a second. Honestly, breaking the habit of reflexively hitting ESC in-between each prototype is harder than learning the language itself.

SkookumScript is a [free plugin](http://skookumscript.com/download/) available on the Unreal Engine marketplace, I'd recommend giving it a try.

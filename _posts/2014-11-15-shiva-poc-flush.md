---
title: 11 Random ShiVa Projects
author: error454
layout: post
permalink: /2014/11/15/shiva/poc/flush/
categories:
  - shiva
tags:
  - shiva
  - poc
  - philanthropy
---

I needed to free up some disk space so I started going through my project folder of about 85 projects. I'll be the first to say that I make a lot of junk and most of it gets thrown away. The good stuff gets thrown into repos.

In my cleaning frenzy, I stumbled upon 11 old shiva projects that sit somewhere between junk and repo status. So I figured, why not release them to the world for dissection, study and ridicule :) These are raw proofs of concept, so expect the code to be fast and loose!

The 11 projects can be [found in my POC repository](https://github.com/error454/ShiVa-Proof-Of-Concept), here is a brief introduction to each of these awkward unfinished things.

## hello-space-invaders ##
<img src='{{ site.url }}/assets/uploads/2014/11/hellospace.jpg' alt='Space Invaders'>

<!--more-->
There are 2 game projects in here, **lander** and **space**. 

### lander ###
lander does basically nothing and I'm not sure what I was thinking when I made it. 2 player space invaders?

### space ###
space is a pretty standard start to space invaders. You can move left/right and shoot but the enemies have no AI.

## FortunousWheel ##
<img src='{{ site.url }}/assets/uploads/2014/11/fortune.jpg' alt='The Fortunous Wheel'>

At one point, the wife and I were doing a motivational chore thing and at the end of the week we would get a prize if we did all of our chores. I made this project so that we could have a Wheel of Fortune style prize picker. So we'd spin the wheel and pull out a numbered piece of paper from a jar telling us what we'd won!

I was really proud of this project because the code to generate the wheel was kind of ridiculous. I do remember the biggest drawback was that the wheel took a TON of draw calls. But the wife had just gotten the HTC One and that thing just handled it so I shrugged and called it good.

## Platformer Physics ##
<img src='{{ site.url }}/assets/uploads/2014/11/platformer.jpg' alt='Platformer Physics'>

A familiar face for a familiar problem. Jumping physics, early jump termination and stuff.

## shiva-pong ##
<img src='{{ site.url }}/assets/uploads/2014/11/pong.jpg' alt='Pong'>

An incomplete pong implementation. Paddle moves, ball moves but the physics need work. Spacebar launches ball, mouse moves paddle.

## random ##
<img src='{{ site.url }}/assets/uploads/2014/11/random.jpg' alt='Random vs Gaussian'>

This project was made to try and visualize the distribution of gaussian random numbers vs random numbers. Try changing the initial state from Gaussian to Random to see for yourself.

## flocking ##
<img src='{{ site.url }}/assets/uploads/2014/11/notflocking.jpg' alt='A steering behavior demo'>

This may have had visions of glory to be a flocking simulator but it ended up just being a steering implementation. Move the mouse around and the box follows it.

## simplex ##
<img src='{{ site.url }}/assets/uploads/2014/11/cottoncandy.jpg' alt='A simplex noise generator'>

This is a shiva plugin and project. The plugin is a simplex noise generator and the project uses it to make random nebulas. The nebulas are rendered to a rendermap and in the `makeRandomNebula` case they are saved to D:\nebula. There was a stretch of time where I was letting this run overnight and I'd come back to a bunch of nebulas in the morning.

I even saved some of my favorite generation parameters to make the nebulas Cotton Candy and Purple Thunder :) We used this at one point to make backdrops for Rage Runner.
 
## letterRender ##
<img src='{{ site.url }}/assets/uploads/2014/11/letterrender.jpg' alt='A 3d space letter generator'>

Hmmm, I can't really say what my motivation behind this was. I think I was inspired by the Wii news browser because they rendered each letter individually and had them animate onto the page. I wanted to try something similar.

## spatialization ##
<img src='{{ site.url }}/assets/uploads/2014/11/spatialization.jpg' alt='A sound spatialization test'>

A simple test with dials for sound spatialization. This also contains code for what I call DebugHUD which is a quick way of getting dials on the screen to tweak AIs.

## HorseTest ##
<img src='{{ site.url }}/assets/uploads/2014/11/horsetest.jpg' alt='Horse Test'>

The original goal of this project was to see if I could take a free unity asset and import it into Shiva. The answer was yes. The next question was how difficult it was to manually modify the horse skeletal system to improve the horse turning animation.

Things got progressively worse as I started playing with a day/night system and then gave up. The horse controller is pretty solid and I'm happy with that small piece of work.

## lua-performance ##
I like for equations to make sense when you read them in code. In most cases, I prioritize readability over imaginary performance. Someone made the comment that dividing is really slow in LUA and I didn't believe them. So I wrote this small test to try and benchmark multiplication and table pre-allocation.

The numbers are so small in my test that it didn't matter.

I mean, technically we know that multiplication is faster than division on the CPU. But can we even perceive it and does it even matter for most of our code? I'd say no for 99% of our code.

As a game designer, I wanted to answer this question for LUA because for me, division vs multiplication is a matter of readability. So here we go down the rabbit hole.

The first step was to see what the LUA VM instructions for multiplication and division were in the version of LUA that shiva was using (5.0.2).

I start with the following lua app:

    local a = 123452.245 / 2.0
    local b = 123452.245 * 0.5


I then compile this and list the LUA machine code:

    error454@olympos:~$ luac -l test.lua
    
    main <test.lua:0> (3 instructions, 24 bytes at 0x144f910)
    0 params, 2 stacks, 0 upvalues, 2 locals, 2 constants, 0 functions
            1       [1]     DIV             0 250 250       ; 2 2
            2       [2]     MUL             1 250 251       ; 2 0.5
            3       [2]     RETURN          0 1 0

You can see that both divide (DIV) and multiply (MUL) are a single LUA instruction.

Let's dive down to the next layer and pop open the LUA source.  In the LUA virtual machine code (lvm.c) we find the implementations for the 4 primary arithmetic operations:

    static void Arith (lua_State *L, StkId ra,
                       const TObject *rb, const TObject *rc, TMS op) {
      TObject tempb, tempc;
      const TObject *b, *c;
      if ((b = luaV_tonumber(rb, &tempb)) != NULL &&
          (c = luaV_tonumber(rc, &tempc)) != NULL) {
        switch (op) {
          case TM_ADD: setnvalue(ra, nvalue(b) + nvalue(c)); break;
          case TM_SUB: setnvalue(ra, nvalue(b) - nvalue(c)); break;
          case TM_MUL: setnvalue(ra, nvalue(b) * nvalue(c)); break;
          case TM_DIV: setnvalue(ra, nvalue(b) / nvalue(c)); break;
     //...ommitted for brevity
    }

We'll work from the outside in.  

There is an important struct for `TObject`, it stores a LUA type (number, string, etc?) and the value of that type.

    typedef struct lua_TObject {
      int tt;		// Type
      Value value;  // Value
    } TObject;

The value of `TObject` has some house-keeping items along with the actual numeric value of the LUA variable:

    typedef union {
      GCObject *gc;
      void *p;
      lua_Number n; // number value
      int b;
    } Value;

Now we can see that `setnvalue` simply sets the type and value of a `TObject`

    #define setnvalue(obj,x) { 
    	TObject *i_o=(obj); 	//interpret object passed in as pointer to TObject
    	i_o->tt=LUA_TNUMBER; 	//Set type to LUA_TNUMBER
    	i_o->value.n=(x); 		//Set value to x (arg)
    }

OK, so `setnvalue `is simply going to take the stack register passed to it and store a number. In our case either:

    nvalue(b) * nvalue(c)
    nvalue(b) / nvalue(c)

That's it?  Better find out what `nvalue` does first to keep our noses clean.

We find the macro to be this pile of junk:

    #define nvalue(o)       check_exp(ttisnumber(o), (o)->value.n) // returns the number value from lua_TObject->value
    
    #define check_exp(c,e)  (e)	//returns e ?!
    #define ttisnumber(o)   (ttype(o) == LUA_TNUMBER) //TRUE if lua_TObject->tt is LUA_TNUMBER
    #define ttype(o)        ((o)->tt) //returns type from TObject->tt

In chess, this would be notated with an exclamation point. Was this an implementation error?  This macro is a complete waste of space, I can only guess that it is unfinished. Maybe a place-holder for eventually checking whether the argument is a number and maybe throwing an assert.

We've reached the end. We can now say with certainty that multiply and divide operations in LUA are straight-up multiply and divide operations in C. So at least we can move the argument from LUA to C.


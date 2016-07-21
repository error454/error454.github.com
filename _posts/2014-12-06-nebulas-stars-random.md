---
title: Nebulas, Stars, Random Numbers and UE4
author: error454
layout: post
permalink: /2014/12/06/nebulas/stars/random/
categories:
  - nebula
  - star
  - gamedev
  - ue4
tags:
  - nebula
  - star
  - ue4
  - perlin
  - simplex
  - material
---

For the last week I've been catching up on ways to render stars, nebulas and space in general. The learning curve is quite steep and I've absorbed enough to finally feel like the firehose has been turned off.

Here is a reflection on what I've learned, some things I've tried and where I might look next when I revisit this.

# Stars #
The first thing I studied about stars was how photoshop people were making cool space scenes. All of these compositing tricks seem very similar:

1. Make a bunch of dots using noise
2. Mask out sections of the noise to make the scene more interesting (less is more)
3. Adjust brightness/contrast so that stars have varying brightness levels

I thought this would be pretty simple to do with a combination of material and blueprint, so here we start with the brute force solution.
<!--more-->

## Step 1 - Spawn Stars ##
I thought it would be nice for level designers to be able to drag a volume into the scene, set the number of stars they desire and see stars pop-up. So I made a blueprint that lets you do this.

<img src='{{ site.url }}/assets/uploads/2014/12/starvolume.jpg'>

I'm using Bluetilities to do this, otherwise you'd have to wire in all your creation on the start of the level. You can even set a seed value so that you get the same star pattern every time. Below you can see that the components of the StarVolume are the volume itself and an instanced mesh that uses the EditorPlane for a static mesh.

<img src='{{ site.url }}/assets/uploads/2014/12/starcomponents.jpg'>

When you run the Respawn function, the blueprint spawns X stars at random points contained inside the box extent. The stars are spawned as instanced static meshes so that the end result is 1 draw call. Here you can see a volume with stars spawned in it.

<img src='{{ site.url }}/assets/uploads/2014/12/flythru.gif'>

## Step 2 - Material ##
The material could probably use some work, here are the things that are working well.

### Meshes Face the Camera ###
I had an issue with the instanced static meshes and getting their actual world position, resulting in some pretty odd stuff. This below is Math Hall example 2.21 from the Content Examples, the only difference is the red outline where I swapped out `ActorPosition` for `ObjectPivotPoint`.

<img src='{{ site.url }}/assets/uploads/2014/12/starworldpositionoffset.jpg'>

### Stars Fade as they Approach Camera ###

The second piece of this is fading the stars out as they approach the camera.

<img src='{{ site.url }}/assets/uploads/2014/12/starfade.jpg'>

### Star Shape ###
You'll notice that for my star shape I'm having a hard time deciding between a DiamondGradient and a RadialGradientExponential.

Here's the two side by side, enlarged for example.
<img src='{{ site.url }}/assets/uploads/2014/12/diamondgradient.jpg'><img src='{{ site.url }}/assets/uploads/2014/12/radialgradient.jpg'>

Basically, use DiamondGradient to get a pointy looking star. The diamonds need to be about twice as large as the radial to get the same effect. Also, things seem a bit dim, so I'd recommend multiplying a bit to get a true white in order to negate the dimming effect of the gradient.

## Step 3 - Material Misc ##
This is a Translucent material right now and I've found that it's important to enable adaptive anti-aliasing. Without this turned on, you kind of lose the stars when the camera moves.

<img src='{{ site.url }}/assets/uploads/2014/12/adaptiveaa.jpg'>

## Final Thoughts on Stars ##
Things I'm happy with

* 3D stars that I can fly through w/free parallaxing and all that
* 1 draw call
* Easy for level designers to use

Things I'm not happy with

* They look like crap compared to artistic space shots that I've seen :(
* No subtle twinkling
* Random distribution is too random-number generator-ish, I'd like to use perlin noise for a more natural star distribution
* Stars in the same volume can only be 1 color. I haven't spent the time on this, but when you create an instanced static mesh, the creation call doesn't pass back that instance. So I'm not sure how to update the dynamic material parameter per-instance... maybe you can't, but that's the first thing to look into. We can always overlay 2-3 layers for multiple colors.

# Nebulas #
A week ago I thought I'd naively go where game developers weren't going by making some great looking volumetric nebulas. Lol! Here's what I can tell you.

## The Big Boys ##
[This paper](https://www.cs.uaf.edu/~genetti/Research/Papers/EG00/Orion.html), written in 2000 helped lay a solid foundation for Emission Nebulas. I'd like to give these guys a standing ovation and an invitation to dinner. The problem with the paper as it applies to games is that they use volumetric rendering. Here is what I understand of their process:

1. Create a surface model of the nebula
2. Calculate a distance field for the surface model - this is basically voxelizing the surface model of the nebula so that they can render not just the surface but some distance (the thickness of the ionization layer of the nebula) under the surface
3. Apply turbulence to the distance field
4. Render top voxels fully transparent and vary the transparency with distance into the ionization layer

As a future note, the formula for [procedural turbulence that they reference](https://design.osu.edu/carlson/history/PDFs/p287-perlin.pdf) is:

    function turbulence(p)
        t = 0
        scale = 1
        while ( scale > pixelsize )
            t += abs(Noise(p/scale) * scale)
            scale /= 2
        return t

Before I fully understood how they were rendering these nebulas, I got prematurely excited because I knew that UE4 used distance fields for a couple features. You can even turn the visualization of distance fields on in the editor. The key to understanding here is surface vs volumes, we render surfaces, the big boys render volumes.

For future use, here is my Nebula 101.

## Nebula 101 ##
A cloud of ionized gas that emits light of various colors. 1 or more nearby stars do the ionizing.

### Most Common Gas Types ###
90% Hydrogen

10% (Helium, Oxygen, Nitrogen)

### Color ###
The color of a nebula is based on the gas type that it is composed of. Light works the way it always does with visible light being emitted by electrons as they jump between different orbitals.

Since emission nebulas are composed mostly of hydrogen, we can use the Balmer (not the Microsoft dude) series to calculate the visible spectrum of light. For Hydrogen, this is only 4 visible wavelengths - red, aqua, violet and... violet which are also known by their much cooler nicknames of H (for hydrogen) followed by a greek letter of the alphabet (alpha,beta,gamma,delta) going from right to left in the photo:

<img src="http://upload.wikimedia.org/wikipedia/commons/6/60/Emission_spectrum-H.svg">

These are wavelengths in nm:

Red (H-Alpha): 656.3

Aqua (H-Beta): 486.1

Violet (H-Gamma) 434.1

Violet (H-Delta) 410.2

Converting these wavelengths to RGB using [this code](https://gist.github.com/error454/a97b9947f6681804b9c6), we get the following 4 colors. I've set the colors up here with their layers screened to see all the possible interactions between them.

<img src='{{ site.url }}/assets/uploads/2014/12/hydrogen.jpg'>

We can see that hydrogen nebulas would be dominated by red at the lowest energy state and shades of blue and a bit of violet. We can easily find the visible spectra for the other most common gas types, Helium and Nitrogen are interesting because they both have an organgish color component.

The bottom line is that if you stick with this color spectra, the nebulas will be believable. A rainbow nebula *could* be cool but it would be much cooler if it had a story behind it with gasses that made sense.

## Nebula Attempts in UE4 ##

### Middleware ###
My first port of call was to check out the middleware [TrueSky](http://simul.co/truesky/). From a conversation with them, it sounds like nebulas would be possible, but the functionality isn't built in right now. Since I'm not making **Nebula: The Game**, I'm not interested in spending time hacking around with their source at this point. Maybe they'll surprise me in the new year with an implementation.

### Building On The Cloud Example ###
There's a nice looking cloud particle in the Content Examples Effects map. I started with this, got rid of the motion, maxed out the lifetime and replaced the cloud texture with some perlin noise.

The thing about perlin noise is that you can play with it forever and get lots of interesting stuff. It's a slow process too because every time you change noise values you have to wait for shaders to recompile. At the points where the noise actually looks good in game, the node preview window doesn't show you anything useful. 

In the end, my best result is still kind of meh, clouds in space. 

<img src='{{ site.url }}/assets/uploads/2014/12/perlinblue.jpg'>

Here is a starting point for this method. The additive material.

<img src='{{ site.url }}/assets/uploads/2014/12/nebulamat.jpg'>

Here is what I call my Perlin base. It's what I start with.

<img src='{{ site.url }}/assets/uploads/2014/12/perlinbase.jpg'>

Want more fluffy clouds? Decrease the scale, here's 0.002 with a level scale of 1.5.

<img src='{{ site.url }}/assets/uploads/2014/12/fluffier.jpg'>

### Other Tests ###
I tried switching the material to lit and creating light/dark areas through lighting.

<img src='{{ site.url }}/assets/uploads/2014/12/nebulalit.jpg'>

I like this workflow for artists and think that the ideal workflow would be the ability to drag out some sort of static meshes into the scene and then light them statically. Particles *sound like a good fit* but they give you very little artistic control due to the particle spawns being random.

Going forward I'd like to move towards a more artist friendly workflow that doesn't involve tuning perlin noise and particle emitter counts. That said, this was a fun diversion, back to making games!

<img src='{{ site.url }}/assets/uploads/2014/12/screenshot.jpg'>



 

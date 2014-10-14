---
title: PhysX 3.3.1 Integration Notebook
author: error454
layout: post
permalink: /2014/06/29/physx-3-3-1-integration-notebook/
categories:
  - Graphics
  - PhysX
  - Shiva 3D
tags:
  - cloth
  - debug
  - eRIGID_STATIC
  - fetchResults
  - filter shader
  - jitter
  - nvidia
  - physx
  - PxClothMeshDesc
  - PxFilterFlags
  - PxGetFilterObjectType
  - PxPairFlag
  - rigid body physics
  - shiva
  - simulate
  - simulation
  - sub-stepping
---
Changelog

July 29, 2014  Updated Rigid Body Section  
July 24, 2014  Added Rigid Body Physics  
July 9, 2014  Added details on collision spheres  
July 2, 2014  Added more cloth tearing info and cloth scale section.  
June 29, 2014  First entry

> Dear Zach, In case of time machine, read this before you start implementing the PhysX SDK. Sincerely, Zach

# Getting Error Messages & Feedback

There are 2 things that would have saved me a few weeks of work had I enabled them from the start.

## PhysX Visual Debugger

PhysX Visual Debugger (PVD) is #1. It's dead simple to get setup following the Nvidia example and it provides you invaluable feedback. Mostly its value is to make sure that *what you think* is happening *is actually what's happening*. Look at the physics objects in PVD alongside with your game, compare the cloth particle count and placement in your game engine along with the PVD, they should sync up.

<a href='{{ site.url }}/assets/uploads/2014/06/pvd.jpg'><img src='{{ site.url }}/assets/uploads/2014/06/pvd-1024x520.jpg' alt='PhysX Visual Debugger'></a>



## Run the Checked Build

You should always run the checked build. If you're working with PhysX in a plugin system, you'll need to attach the debugger to your main process. Doing so will let you see the assert messages that pop up in the Visual Studio console. They will tell you when you do stupid things, like pass bogus (but not invalid) parameters to class instantiations. You might otherwise spend a lot of time figuring out why your physics objects are acting oddly.

# Cloth

## PxClothMeshDesc Weights

Of course you understand that these weights determine whether the cloth particle is simulated or static. 0 means the particle is static, > 0 means that the particle is simulated. Usually we just use 0.0f and 1.0f to distinguish these two. A key part of setting up the cloth correctly is making lists of the vertices that need to be frozen in place vs simulated.  You'll need to write a few custom tools that allow you to select these vertices and save them out as a list so that you can populate these weights at runtime. Don't bother trying to split up your vertices and cloth weights into separate variables. I originally tried stuffing the weights into their own array and would continuously get the error:

<pre><code>..\..\PhysX\src\NpFactory.cpp (427) : invalid parameter : PxPhysics::createCloth: particle values must be finite and inverse weight must not be negative</code></pre>

Despite the fact that my particle values were finite in length/value and all my inverse weights were non-zero. Eventually I gave up and followed the nvidia sample code where they stuff the vertices and weights into a single PxVec4 like a thanksgiving day turkey. Vertices go in (x,y,z) and weight goes in (w).

## Cloth Is Tearing

You may get excited that you've found a new feature to allow tearing of cloth when half your flag just up and jumps off the flagpole. If your cloth is falling apart like this. It probably means that your mesh has duplicate vertices. In my case, the tessellated plane had duplicate vertices that needed to be welded.

<a href='{{ site.url }}/assets/uploads/2014/06/tear.jpg'><img src='{{ site.url }}/assets/uploads/2014/06/tear-1024x525.jpg' alt='Cloth Tearing'></a>

There is at least 1 other case that causes tearing. The problem is that not all duplicate vertices are geometric. In my case, some duplicate vertices in other meshes were part of the UV map. If any of these vertices are set to simulated, the cloth will have issues. Since I wasn't able to separate out the UV from the mesh vertices, I wrote a quick check for duplicate vertices and made sure that they were forced to a weight of 0.0f.

<a href='{{ site.url }}/assets/uploads/2014/06/duplicate-verts.jpg'><img src='{{ site.url }}/assets/uploads/2014/06/duplicate-verts-1024x579.jpg' alt='Red vertices are duplicates'></a>

## Cloth Scale

The scale of the cloth can be tricky. When you are slurping down the vertice translations info for **PxClothMeshDesc**, you need to multiply each vertice by the relevant scale factor of the object so that the space representation in PhysX is the same as in the engine. The problem comes in during the update phase when you're matching the vertices in engine with the vertices as PhysX delivers them. I soon discovered that after updating the vertices in engine, the engine was applying the original scale factor to them! This results in a double scaling.

It's pretty obvious what's going on here. The coordinates in the engine are local space and get a scale factor applied. One option is to apply a reverse scale factor in the vertice update function. This has a huge downside in that to do this you have to transform every individual vertice each frame. In my case, I'm using a single function call to update my mesh vertices which has speed advantages. So, the simple solution is to:

1.  Create PxClothMeshDesc with scaled vertices
2.  Set the object uniform scale to 1

This workaround gives PhysX the correctly scaled vertices and then relies on the PhysX representation for future updates. Since the uniform scale is then essentially zero'd out (at 1), no further scale factor is applied to mesh vertice updates!

## Cloth Collision Spheres

Collision sphere positions are defined in the Local Space of the cloth object! When defining the collision spheres that make up each capsule for a character, it made more sense to define those in terms of local space offsets from each bone. Then once you know the offset from the character root to the cloth root, you can apply the necessary transforms to update the sphere positions each frame.

<a href='{{ site.url }}/assets/uploads/2014/06/collision-sphere.jpg'><img src='{{ site.url }}/assets/uploads/2014/06/collision-sphere-1024x576.jpg' alt='Collision sphere&apos;s are defined in local space of the cloth root.'></a>

A small sidenote here is that using a boxy model with spherical colliders does not look very accurate when simulated.

# Rigid Body Physics

## Filter Shader for Rigid Static Collision

When using a custom filter shader, filter masks seem to be completely ignored on rigid static objects. <a href="https://devtalk.nvidia.com/default/topic/763481/physx-and-physics-modeling/why-don-t-filter-flags-work-for-static-objects-physx-3-3-1-windows/" target="_blank">I've asked why</a>. Here is my current filter shader that handles rigid static collisions correctly.

<pre><code>PxFilterFlags MyFilterShader(PxFilterObjectAttributes attributes0, PxFilterData filterData0, 
                            PxFilterObjectAttributes attributes1, PxFilterData filterData1,
                            PxPairFlags& pairFlags, const void* constantBlock, PxU32 constantBlockSize)
{
    // let triggers through
    if(PxFilterObjectIsTrigger(attributes0) || PxFilterObjectIsTrigger(attributes1))
    {
        pairFlags = PxPairFlag::eTRIGGER_DEFAULT;
        return PxFilterFlag::eDEFAULT;
    }

    // generate contacts for all that were not filtered above
    pairFlags = PxPairFlag::eCONTACT_DEFAULT;

    // Check for static object collision
    if( PxGetFilterObjectType(attributes0) == PxFilterObjectType::eRIGID_STATIC || PxGetFilterObjectType(attributes1) == PxFilterObjectType::eRIGID_STATIC )
    {
        pairFlags |= PxPairFlag::eNOTIFY_TOUCH_FOUND | PxPairFlag::eNOTIFY_CONTACT_POINTS ;
        return PxFilterFlags();
    }

    // trigger the contact callback for pairs (A,B) where
    // the filtermask of A contains the ID of B and vice versa.
    if((filterData0.word0 & filterData1.word1) && (filterData1.word0 & filterData0.word1))
    {
        pairFlags |= PxPairFlag::eNOTIFY_TOUCH_FOUND | PxPairFlag::eNOTIFY_CONTACT_POINTS ;
        return PxFilterFlags();
    }

    // If the mask didn't match, then suppress the collision
    return PxFilterFlag::eSUPPRESS;
}</code></pre>

## Reducing Simulation Jitter

I found sub-stepping to greatly reduce jitter in my simulations involving kinematic actors.

### Sub-Stepping

<pre><code>int substeps = 3; // Modify number of substeps based on your needs
float subStepSize = mStepSize / substeps;
for ( PxU32 subStepI = 0; subStepI  substeps; subStepI++ )
{
    gScene-simulate(subStepSize);
    gScene-fetchResults(true);
}</code></pre>

Also, thanks to Gordon at Nvidia, I discovered that I was <a href="https://devtalk.nvidia.com/default/topic/763104/physx-and-physics-modeling/dynamic-kinematic-collision-jitter-on-physx-3-3-1-win-64-bit/" target="_blank">not setting the moment of inertia </a>on my actors. This was the main factor causing jitter and is also very noticeable because it causes objects to have very high moments of inertia which means it takes far less torque to get them spinning.

---
title: Using Animation Blueprints with Matinee
author: error454
layout: post
permalink: /2015/01/30/matinee/animation/blueprint
categories:
  - ue4
tags:
  - ue4
  - matinee
  - anim
  - bp
---

I recently did some matinee scenes in UE4 and came away feeling like I was using the tools wrong. That's probably because I'm a programmer with no animation skills trying to use tools meant for artists with no programming skills :o Add on top of this a time constraint and you have a somewhat willy nilly solution that I'm about to describe.

What follows is a summary of how I worked around the tool limitations to arrive at a solution that worked for my skillset. One of the end results can be seen below:

<iframe width="560" height="315" src="https://www.youtube.com/embed/ToAmkclHzWU" frameborder="0" allowfullscreen></iframe>

<!--more-->

# What Matinee Wants #
Matinee seems to want a raw skeletal mesh in your level. It then lets you move this mesh and trigger animations. The intent seems to be that you're either going to provide a big custom animation for your entire sequence, or your animation clips are done in such a way that they already blend between one another. Matinee **does not blend**! I can't emphasize this enough, you cannot blend between animation tracks in Matinee!

This whole not blending thing may come as a huge surprise to many people. The challenge for me was that I didn't have a bunch of custom animations. The assets I had to work with were:

* 1 Animation track (idle)
* 3 Aim poses for each arm (used for aim blendspace)
* 3 Look poses for the head (used for head look blendspace)
* A handful of custom poses (1 frame, no animation)

# Why Not use an Animation Blueprint? #
I thought to myself, hmm, what lets me trigger animations and also does animation blending, IK and stuff? Obviously the answer was animation blueprints! This excited me because I had already solved the puzzle of animation blueprints and I knew they did exactly what I wanted, I could move my characters head at a target and move his arms up and down to aim at stuff.

## Matinee Destroys Animation Instances ##
Any skeletal mesh that has a matinee track will have their animation instance ripped out and thrown away when the matinee starts. This means that you cannot have a skeletal mesh that is both using an animation blueprint AND part of a matinee track. I wrote-up an [answerhub issue](https://answers.unrealengine.com/questions/164530/skeletalmeshactor-cant-use-animation-bp-matinee.html) on this although I think the limitation is probably by design.

# How to Use Animation BP in Matinee #
This setup may seem odd at first, so here is a high-level overview of how it works.

<img src='{{ site.url }}/assets/uploads/2015/01/matinee-overview.jpg'>

There are 3 total components here:

1. The Star
2. The Protege
3. The Animation Blueprint

## The Visible Animated Actor (The Star) ##
This can be a blueprint or a skeletal mesh actor, it doesn't matter. This is going to be the visible actor in your matinee sequence. Every time you want to play an animation, you'll instead fire an event from matinee, that event will enable a state in your animation blueprint to enable the desired animation.

To hook up the star, when the matinee starts you need to attach him to the protege via some code or blueprint like:

    AttachRootComponentToActor(MatineeActorToMatch, NAME_None, EAttachLocation::SnapToTarget, false);

If you aren't a coder you can skip the rest of this section and move on to the next heading. One thing I wanted is smooth entry into a matinee sequence. When a matinee begins, I don't know where the player character is, they might be facing the wrong direction or slightly off target from where my matinee actor begins. So here is really kind of a full peak at how I'm trying to manage this.

First I enter matinee mode with an optional time until the transition is locked:

    void ARob1ePawn::EnterMatineeMode(float TimeUntilTranslationLock, AActor* ActorToMatch)
    {
    	DisableInput(UGameplayStatics::GetPlayerController(GetWorld(), 0));
    	bIsInMatineeMode = true;
    	MatineeTimeUntilTranslationLock = MatineeTimeMax = TimeUntilTranslationLock;
    	MatineeActorToMatch = ActorToMatch;
    }

Now when my pawn ticks, I lerp the rotation and location over the specified time period and when the time expires, I attach to the matinee actor:

    void ARob1ePawn::Tick(float DeltaSeconds)
    {
    	Super::Tick(DeltaSeconds);
    
    	/***
    	*     __    __     ______     ______   __     __   __     ______     ______
    	*    /\ "-./  \   /\  __ \   /\__  _\ /\ \   /\ "-.\ \   /\  ___\   /\  ___\
    	*    \ \ \-./\ \  \ \  __ \  \/_/\ \/ \ \ \  \ \ \-.  \  \ \  __\   \ \  __\
    	*     \ \_\ \ \_\  \ \_\ \_\    \ \_\  \ \_\  \ \_\\"\_\  \ \_____\  \ \_____\
    	*      \/_/  \/_/   \/_/\/_/     \/_/   \/_/   \/_/ \/_/   \/_____/   \/_____/
    	*
    	*/
    	if (bIsInMatineeMode)
    	{
    		if (MatineeTimeUntilTranslationLock >= 0.f && MatineeActorToMatch)
    		{
    			MatineeTimeUntilTranslationLock -= DeltaSeconds;
    
    			FVector MatchLocation = MatineeActorToMatch->GetActorLocation();
    			FRotator MatchRotation = MatineeActorToMatch->GetActorRotation();
    
    			// Attach actor
    			if (MatineeTimeUntilTranslationLock <= 0)
    			{
    				SetActorLocationAndRotation(MatchLocation, MatchRotation);
    				AttachRootComponentToActor(MatineeActorToMatch, NAME_None, EAttachLocation::SnapToTarget, false);
    			}
    			else
    			{
    				// Match Translations over time period
    				FVector Location = GetActorLocation();
    				FRotator Rotation = GetActorRotation();
    
    				Location = FMath::Lerp(Location, MatchLocation, 1 - MatineeTimeUntilTranslationLock / MatineeTimeMax);
    				Rotation = FMath::Lerp(Rotation, MatchRotation, 1 - MatineeTimeUntilTranslationLock / MatineeTimeMax);
    
    				SetActorLocationAndRotation(Location, Rotation);
    			}
    		}
    		return;
    	}
        ...

Of course when you're all done you should detach from the matinee actor and re-enable input!

## The Matinee Skeletal Mesh Actor (The Protege) ##
This skeletal mesh will be the actor that you move around in matinee. The actor will be set to hidden in the game because he's really just around as a placeholder. Your visible actor will be attached to this actor so that he follows the location and rotation that you are animating.

The problem with triggering events that play animations on your animation blueprint is that now matinee scrubbing doesn't show the animations in the scene. For this reason I recommend triggering the animations on this matinee track just for your own reference while scrubbing the matinee sequence.  

## Custom Animation Blueprint ##
You will probably want to make a custom state for your matinee work as I've done here. My main goal was to be able to blend my idle animation with basically everything and to trigger specific states with very specific in/out transition times.

<img src='{{ site.url }}/assets/uploads/2015/01/matinee-customstate.jpg'>

The anim bp itself is mostly a bunch of **Blend Poses by bool** and a few additive nodes.

<img src='{{ site.url }}/assets/uploads/2015/01/matinee-animgraph.jpg'>

## Putting it all Together ##
Your matinee sequences will start to look like this.

<img src='{{ site.url }}/assets/uploads/2015/01/matinee-skeletaltrack.jpg'>

Notice how I fire the `RobGunCheck` event and the `Rob_gun_tap` animation at the same time. I didn't have to fire that animation, but it sure helps because now I can scrub matinee and also have a perfect timeline for when the animation will end. Continuing forward, you can see I fire `RobLookLeft` and `Lookatdog` at the same time etc.

At the end of the day, your matinee controller is going to look like this.

<img src='{{ site.url }}/assets/uploads/2015/01/matinee-events.jpg'>

Where each of these events in a nutshell only needs to set a bool value in your animation blueprint like so.

<img src='{{ site.url }}/assets/uploads/2015/01/matinee-levelbp.jpg'>

# Summary #
You can combine matinee and animation blueprints using the above workflow. Since events don't fire when scrubbing matinee, you will need to also add the skeleton animation track for reference even though it's the animation blueprint that fires the animation in the final sequence.

Ideally in the future, we'd be able to blend between animation tracks in matinee and also get more granular skeletal control, for instance an IK rig. I'll admit, this workflow may seem odd, but it works for my skills. If you have a similar workflow that you think might work better, I'd love to hear it! 
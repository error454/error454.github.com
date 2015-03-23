---
title: Character Movement Replication in UE4
author: error454
layout: post
permalink: /2015/03/20/ue4/movement/replication
categories:
  - ue4
tags:
  - ue4
  - c++
  - network
  - replication
  - character movement
---

<img src='{{ site.url }}/assets/uploads/2015/03/net-corrections.jpg'>

There's a [piece of documentation](https://docs.unrealengine.com/latest/INT/Gameplay/Networking/CharacterMovementComponent/index.html) on the official UE4 site that briefly discusses different methods for replicating Character Movement over the network. It explains 4 different methods for replicating new movement  abilities and goes on to detail why 3 of the methods basically just kind of suck.

The short explanation is that the preferred method not only keeps the client synced but can also replay moves that would pile up during a lag spike. This way, any movement based abilities triggered by the client during the spike won't get lost.

What follows is a concrete example of Approach 4 that is mentioned in the document. This took me several days to nail down, most of the major structure was discovered by looking at the Unreal Tournament source. There you will find an implementation for sliding, dodging and sprinting. The downside of the UT source is how huge it is! I hope this serves as a good basis to build your movement based replication with!

<!--more-->

## Resources ##

* [Original Documentation](https://docs.unrealengine.com/latest/INT/Gameplay/Networking/CharacterMovementComponent/index.html)
* [My Sample Project](https://github.com/error454/CharacterMovementReplication-UE4)

## When to use this Method? ##
It should be clear when to use this method vs running authoritative methods on the Server. This method is only used for abilities that somehow affect character movement. By default, the engine handles Jumping and Crouching using this method.

You wouldn't shoot a gun or activate an airstrike using this method, instead you'd call those directly on the server. But if you're adding a double jump, sprint or teleport (all abilities that directly affect the movement of the character) then this is the right method to use.

## The Sample Project ##
I wanted to make my additions to the original Side Scroller sample crystal clear. To do so I've split up my code into separate commits. The first commit is the original Side Scroller C++ template, [another commit](https://github.com/error454/CharacterMovementReplication-UE4/commit/deb9509d41b22cec810e5a9e6749fe3543298292) illustrates the minimum necessary code needed to implement a sprint ability. You can look at the commit log and see all of the changes made quite easily.

## Time Caveats ##
One thing that the UE4 character movement component does is regularly reset client timestamps to maintain a higher accuracy. The function responsible for updating these timestamps is part of the network prediction class that we've overridden and is called `UpdateTimeStampAndDeltaTime` which is called to by a `CharacterMovement` component function of the same name.

I noticed that the UT code adjusts all of the saved move movement timers whenever client time is adjusted in this manner. However, the UE4 codebase does not. Looking at the `CharacterMovementComponent`, it doesn't adjust any timers as a result of a client time change. `JumpKeyHoldTime` is an example of a timer that is stuffed into the saved moves list and not adjusted when a client timestamp is reset.

I also want to call out that in UT code, depending on whether you are client or server, different values are used to represent the current world time (see `GetCurrentMovementTime()`). The two choices are:

* A variable called `CurrentServerMoveTime`
* `CharacterOwner->GetWorld()->GetTimeSeconds()`

If you trace who sets `CurrentServerMoveTime` and what it is set to you'll eventually find that it either ends up taking the client timestamp from the network prediction data or `GetWorld()->GetTimeSeconds()`.

### TLDR; ###
I didn't implement the timer corrections seen in UT because I don't yet fully understand the significance. It seems to me that the delta between time corrections would be sub-millisecond. This area requires more research and probably some debug logging in UT to really nail down. 

## Testing Replays ##
To trigger network replays, I found it helpful to introduce some packet loss into the simulation. [This article](https://www.unrealengine.com/blog/finding-network-based-exploits) explains the flags you need to do so. I used the following console commands to simulate 500 ms (+/- 100 ms) with 15% packet loss:

    p.NetShowCorrections 1
    Net PktLag=500
    Net PktLagVariance=100
    Net PktLoss=15

## Conclusion ##
Character based movement replication and ability replay is, well, kind of involved! Hopefully you get a better start than I did :)
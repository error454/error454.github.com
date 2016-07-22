---
title: Triggering Overlaps for Level Streaming Actors
author: error454
layout: post
permalink: /2016/05/12/overlap/patch/
categories:
  - ue4
  - c++
tags:
  - ue4
  - c++
  - overlaps
  - pull request
  - patch
---
In our game we use streaming levels to load the player into various interior buildings. The player enters an airlock, which is just a loading screen that they can move around in while they wait for the level to load around them.

<img src='{{ site.url }}/assets/uploads/2016/05/airlock.jpg'>

This allows us to keep our interiors in separate maps. One problem that we encountered is that we have some custom volumes that we use to change camera parameters. When the player pawn overlaps a volume, we add it to the list of camera parameter volumes in play and those get sorted and used based on a priority value (similar to how post process volumes work). 

The problem is that when we stream a level in, OnBeginOverlap events don't fire for any intersecting actors. This was a big problem for us since without the overlap, our new camera parameters don't get registered and that left our camera out in no-mans-land, showing all our seems.

<!--more-->

It turns out that [this is by design](https://github.com/EpicGames/UnrealEngine/blob/c07c63dcdedb7e8ced9a81dfb864505d5db5afa3/Engine/Source/Runtime/Engine/Private/Level.cpp#L1771). 

Our use case is one where we want those overlaps to trigger, so what to do? I submitted [pull request 2379](https://github.com/EpicGames/UnrealEngine/pull/2379) that provides a flag for AActor (`bGenerateOverlapEventsDuringLevelStreaming`) that allows specifying that we want to trigger overlaps when that actor is streamed in. This isn't an all or nothing type of thing, it's very specific to actors that the player will be overlapping near the entrance of a level, this change is working perfectly for us.

**Update**
My pull request was accepted on July 21, so should show up in the next release.

As a side note, the curious observer might wonder how level streaming differs from level loading in regards to overlapping actors. The simple answer is that the primary difference between the two is the order of calls to `BeginPlay` and `UpdateOverlaps`.

**Level Streaming**

```
UpdateOverlaps()
BeginPlay()
```

**Level Loading**

```
BeginPlay()
UpdateOverlaps()
```


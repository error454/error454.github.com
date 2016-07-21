---
title: Vive Motion Controller UE4 Bindings
author: error454
layout: post
permalink: /2016/04/20/vive/ue4/bindings/
categories:
  - ue4
  - vr
tags:
  - ue4
  - vr
  - vive
  - motion
  - bindings
  
---
Getting the Vive motion controller buttons and axis bound in UE4 is one of the first steps to playing around in VR.

I had to dig into the [UE4 code](https://github.com/EpicGames/UnrealEngine/blob/dff3c48be101bb9f84633a733ef79c91c38d9542/Engine/Plugins/Runtime/Steam/SteamVR/Source/SteamVRController/Private/SteamVRController.cpp#L105 "UE4 code") a bit to find out what these bindings were. I'm throwing them out in the open here for future reference.

<!--more-->

{% gist f3dfb860629efaa12e954a71521e63cd %}


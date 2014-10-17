---
title: Packaging Unreal Engine Editor
author: error454
layout: post
permalink: /2014/10/16/ue4-packaging/
categories:
  - UE4
tags:
  - UE4
  - editor
  - packaging
  - unreal
  - engine
---

##Packaging Unreal Engine

After building the latest 4.5 editor, I needed to package it up and move it to another machine. Figuring out which folders were required took me awhile, so here is to saving time in the future.

Once the build has completed, pull the following folders out of the **Engine** folder to arrive at the folder structure below:

* /Root
    * Samples   (Optional)
    * Templates (Optional)
    * Engine
        * Binaries
            * DotNET
            * ThirdParty
            * Win64
        * Config
        * Content
        * DerivedDataCache
        * Documentation (Optional)
        * Extras (Optional)
        * Intermediate
        * Plugins
        * Programs
        * Saved
        * Shaders
    
I should mention that I ommitted the Android and IOS folders from the Binaries folder. This would be relevant if you are doing mobile. Also, I was building for 64-bit only, so you may need the Win32 folder or other build binaries for your OS.

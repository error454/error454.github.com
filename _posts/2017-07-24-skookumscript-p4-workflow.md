---
title: SkookumScript Perforce Workflow
author: error454
layout: post
published: true
permalink: /2017/07/24/skookumscript/p4/workflow
categories:
  - ue4
  - blueprint
  - skookumscript
tags:
  - ue4
  - blueprint
  - skookumscript
  - source control
  - perforce
---
This is a quick guide on how to integrate SkookumScript into a P4 workflow for UE4. Based on whether you are a Blueprint engineer, a C++ engineer or both, this guide will show you how to easily work with SkookumScript and keep assets up-to-date for others.

*Versions Used*

* SkookumScript 3.0.5093
* UE4 4.16.2

<!--more-->

# SkookumScript Directories

It's useful to familiarize ourselves with some key directories. SkookumScript stores files in the project's Scripts directory, there you will find a hierarchy like this:

```
Scripts
|-Project
|-Project-Generated-BP
|-Project-Generated-C++
```

### Project

When you create new SkookumScript functionality (scripts and classes), those files end up here.

### Project-Generated-BP

The content in this folder is automatically generated and managed by the SkookumScript plugin. Generation occurs when a blueprint is loaded/compiled in the Unreal Engine editor.

### Project-Generated-C++

The content in this folder is also automatically generated and managed by the SkookumScript plugin. Generation occurs as part of the build process.

# P4 Typemap Recommendation

I'd recommend configuring a typemap for your project so that the auto-generated files are not read-only. As an example, the typemap entries for SpaceInvaders might look like:

```text+w //SpaceInvaders/Scripts/Project-Generated-BP/...```
```text+w //SpaceInvaders/Scripts/Project-Generated-C++/...```


# Blueprint Workflow

When working in blueprints, you need to keep an eye on the `Project-Generated-BP` folder. Any time you add a new function or variable to your BP, the auto-generated contents for that BP will change in `Project-Generated-BP`. 

Once you're ready to check your BP changes in, the easiest way to find/grab the auto-generated changes is to use the P4 feature `reconcile offline work`. Be sure to only perform this on the `Project-Generated-BP` folder by right-clicking on it.

<img src='{{ site.url }}/assets/uploads/2017/07/reconcile.jpg'>


# C++ Workflow

Once you've added new C++, you will need to keep the `Project-Generated-C++` folder in sync. By default these days, this folder is only a single file so it should be easy to keep this updated. Obviously, make sure you pull from P4 before you compile and check-in so that you don't wipe out other C++ changes that might have snuck in after your last pull.

Additionally, if your project is setup where you check in your editor binary to the repo e.g. `Binaries/Win64/UE4Editor-ProjectName.dll` then you will also need to check-in the following additional files with any C++ change:

* `Plugins/SkookumScript/Binaries/Win64/UE4Editor-SkookumScriptEditor.dll`
* `Plugins/SkookumScript/Binaries/Win64/UE4Editor-SkookumScriptEditorGUI.dll`
* `Plugins/SkookumScript/Binaries/Win64/UE4Editor-SkookumScriptRuntime.dll`
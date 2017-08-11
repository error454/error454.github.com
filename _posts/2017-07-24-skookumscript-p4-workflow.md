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
*Updated 8/10/2017*

This is a quick guide on how to integrate SkookumScript into a P4 workflow for UE4. Based on whether you are a Blueprint engineer, a C++ engineer or both, this guide will show you how to easily work with SkookumScript and keep assets up-to-date for others.

*Versions Used*

* SkookumScript 3.0.5279
* UE4 4.16.2/4.17

<!--more-->

# SkookumScript Directories

SkookumScript stores files in the project's Scripts directory, there you will find a hierarchy like this:

```
Scripts
|-Project
|-Project-Generated-BP
|-Project-Generated-C++
```

### Project

When you create new SkookumScript functionality (scripts and classes), those files end up here.

### Project-Generated-BP

The content in this folder is automatically generated and managed by the SkookumScript Plugin. Generation occurs when a blueprint is loaded/compiled in the Unreal Engine editor. Make sure you have Unreal Engine connected to P4 so that the contents here get managed automatically.

### Project-Generated-C++

The content in this folder is automatically generated and managed by the SkookumScript plugin. Generation occurs as part of the build process.

# What Goes Into Source Control?

Projects will fall into 1 of 2 categories based on whether everyone compiles or only devs compile.

### Everyone Compiles

When everyone compiles, check the following folders into P4:

* `Scripts/Project`
* `Scripts/Project-Generated-BP`

FYI: If your project is a Blueprint only project, then everyone compiles.

### Only Devs Compile

If only developers compile (they check-in the editor binary) then check the following into P4:

* `Scripts/Project`
* `Scripts/Project-Generated-BP`
* `Scripts/Project-Generated-C++`
* `Plugins/SkookumScript/Binaries/Win64/UE4Editor-SkookumScriptEditor.dll`
* `Plugins/SkookumScript/Binaries/Win64/UE4Editor-SkookumScriptEditorGUI.dll`
* `Plugins/SkookumScript/Binaries/Win64/UE4Editor-SkookumScriptRuntime.dll`
 
In addition, you may want to add a P4 Typemap for the ```Project-Generated-C++``` folder so that generation doesn't accidentally fail due to the overlay file being read-only. As an example, the typemap entry for SpaceInvaders might look like:

* ```text+w //SpaceInvaders/Scripts/Project-Generated-C++/...```

# Workflow for SkookumScript Code

Be sure that you configure the SkookumScript IDE for Source Control. Once configured, the IDE will automatically check-out and add files to the default changelist as you work directly in SkookumScript.

You simply need to push your changes when finished. If you've made changes without being configured for P4 then you may need to manually reconcile the necessary folders.

# Workflow for Blueprints

When SkookumScript is enabled in a project and you are working with blueprints, things will be automatically generated under the hood in the `Project-Generated-BP` folder. In the latest versions of SkookumScript, the SkookumScript Plugin automatically checks out/adds these auto-generated files to your default changelist. You will want to make sure you have Unreal Engine configured for source control and that you are connected to P4 with an active session.

Any time you add a new function or variable to your BP, the auto-generated contents for that BP will change in `Project-Generated-BP`. Once you're ready to check your BP changes in, be sure to check your default changelist and include any `Project-Generated-BP` changes that are relevant. 

# Workflow for C++

If you fall under the **Everyone Compiles** camp, then there's nothing extra you need to check-in for your C++ workflow. Otherwise, you will need to keep the `Project-Generated-C++` folder in sync. Make sure you pull from P4 before you compile/check-in so that you don't wipe out other C++ changes that might have snuck in after your last pull.

For all workflow types be aware that changes made in C++ will also necessitate changes to the auto-generated contents for any inherited Blueprints. These blueprints will need to be opened in the editor to make sure that the contents in `Project-Generated-BP` are updated. Keep this in mind if you are making a change to a base class that is heavily inherited as you may need to open all of the impacted Blueprints to make sure the auto-generated content is updated.

If **Only Devs Compile** and you check in your editor binary to the repo e.g. `Binaries/Win64/UE4Editor-ProjectName.dll` then you will also need to check-in the following additional files with any C++ change:

* `Plugins/SkookumScript/Binaries/Win64/UE4Editor-SkookumScriptEditor.dll`
* `Plugins/SkookumScript/Binaries/Win64/UE4Editor-SkookumScriptEditorGUI.dll`
* `Plugins/SkookumScript/Binaries/Win64/UE4Editor-SkookumScriptRuntime.dll`
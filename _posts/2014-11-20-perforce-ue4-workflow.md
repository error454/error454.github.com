---
title: How to use Perforce in UE4
author: error454
layout: post
permalink: /2014/11/20/perforce/ue4/workflow
categories:
  - perforce
  - ue4
tags:
  - perforce
  - ue4
  - source control
---

# Intro #
Perforce is a source control repository used to track and control changes to files. You should read the [official perforce docs for UE4](https://docs.unrealengine.com/latest/INT/Engine/UI/SourceControl/index.html) first. Then come back here for more details. The benefits of perforce are:

## Tracking ##
Helps us keep tabs on changes to assets. For instance if an asset breaks, we can look at the history of an asset. Here is the history of our player character:

<img src='{{ site.url }}/assets/uploads/2014/11/filehistory.jpg' alt='File History'>
<!--more-->

If needed, we can revert back to an older version to fix issues. We also have a good list of who modified something and what they modified.

## Control Changes ##
Perforce makes sure that only a single person can make changes to an asset at a time. Imagine what would happen if two people were modifying the same asset simultaneously and then both saved. Whoever saved last would overwrite all the changes of the first person!

# Working in Perforce #
Perforce has essentially 5 actions that you can perform inside the Unreal Engine Editor. These actions are accessed by right clicking on an asset, folder or project root and looking in the Source Control menu:

<img src='{{ site.url }}/assets/uploads/2014/11/rightclick.jpg' alt='Picture of right-click source menu'>

## Check Out ##
Locks the file and allows you to make changes to the file.

## Mark for Add ##
Marks a new file to be added to the repository. You can only mark files that don't already exist in the repo.

## Check In ##
Submits all of your changes to the server and unlocks the file.

You can check in:

* A single asset at a time
* All new/changed assets at once (accessed from **File->Submit To Source Control**)

Always look through the files that are about to be submitted and ask yourself the question

**Question**:  *Did I really change this asset?*

If the answer is no then don’t submit that asset. Revert it instead. Notice below that I've only selected to submit the files that were deliberately changed.

<img src='{{ site.url }}/assets/uploads/2014/11/submitfiles.jpg' alt='Picture of writing changelist and checking the files that were changed.'>

## Revert ##
Reset the asset to the state it was in before it was checked out. Also unlocks the file if it was previously checked out.

## Sync ##
Pulls down the latest version of the file from the server. This gives you all of the latest changes from other collaborators.

You can sync:

* A single asset
* An entire folder

# Perforce Workflows #

## Workflow 1: Updating Code ##
You will typically do this first thing in the morning or when your coder says you need to update code. You need to close down UE4, open the P4V client and Get Latest. Make sure you’ve selected the top level folder for the project first.

<img src='{{ site.url }}/assets/uploads/2014/11/p4v-client-update.jpg' alt='update client'>

Once syncing is complete, you can launch the editor again.

## Workflow 2: Adding New Files ##
<img src='{{ site.url }}/assets/uploads/2014/11/addnew.png' alt='update client'>

## Workflow 3: Making Changes to Existing Files ##
Keep in mind that once an asset is checked out, nobody else is allowed to make changes until you either check in or revert. So don’t leave important assets checked out over the weekend!

<img src='{{ site.url }}/assets/uploads/2014/11/workflow1.png' alt='make changes to existing files'>

## Workflow 4: Syncing Changes ##

<img src='{{ site.url }}/assets/uploads/2014/11/syncworkflow.png' alt='sync files up'>

> Sometimes when syncing content, the Yellow Exclamation will not go away and it will appear that your content isn’t syncing. This is called the **Sync of Death**. Never check out a file that is in the SoD state (if you’ve already checked out the file, revert it). When you have files in the SoD state, you will need to restart Unreal Engine and sync again before working with those files.

# Conclusion #
These are the basics of working with perforce in UE4. At first the workflow may seem counterproductive, but you'll get the hang of it and eventually it will be second nature. The areas where you need to exercise the most caution are the buggy Sync of Death and the accidental Check-In of files that you did not intend to edit.

Keep in mind that it's ok to make new test assets, maps etc if you need to do some testing. You can leave these test assets in your project, just remember to not mark them for add. Just be careful that the assets you are checking in don't depend on any of your test assets! 

Keep those check in descriptions good and your entire team will be happy!
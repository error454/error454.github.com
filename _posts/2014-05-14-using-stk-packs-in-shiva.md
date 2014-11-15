---
title: Using STK Packs in ShiVa
author: error454
layout: post
permalink: /2014/05/14/using-stk-packs-in-shiva/
categories:
  - Shiva 3D
tags:
  - 3d
  - packaging
  - packs
  - shiva
  - shiva3d
  - stk
---
Most ShiVa games have a single STK pack file. The reason why is because it's easy to hit the export button and dump everything into one giant file when you publish. On many PC platforms like Desura and Steam, the client does a proper diff and then only downloads chunks of files that are different than what he already has.  In Rage Runner, our pack file is about 250 MB. In an update where the only thing we've changed is 2 kb of XML, the player still has to download the entire 250 MB pack!

This is obviously wasteful. In this tutorial I am going to detail:
<!--more-->
*   How to create STK files that contain your content
*   How to use STK files in your game
*   How to integrate this into your editor workflow
*   Caveats for publishing on individual platforms



## Get the Sample Project

There is a sample project available below. All of the screenshots and code samples in this article come from this project.

[Download Sample Project][1]

## Creating STK Content

The first step is to create STK files that contain only your content. We are going to make an STK file that has all of our models and materials. To do so:

1.  Select content you want to export
2.  Right-Click and add to export 
    <a href='{{ site.url }}/assets/uploads/2014/05/export1.jpg'><img src='{{ site.url }}/assets/uploads/2014/05/export1-300x235.jpg' alt='Right-click and add to export.'></a> 
    
3.   Continue with step 1 for all assets 
        <a href='{{ site.url }}/assets/uploads/2014/05/export2.jpg'><img src='{{ site.url }}/assets/uploads/2014/05/export2-300x235.jpg' alt='Continue selecting assets and right-clicking -&gt; add to export. Until all items are added.'></a> 
        
4.  Finally click export 
            <a href='{{ site.url }}/assets/uploads/2014/05/export3.jpg'><img src='{{ site.url }}/assets/uploads/2014/05/export3-300x217.jpg' alt='Select your export target platform. Here I select PC.'></a>  
            
            
## Placing Pack Files

When packaging STK files with your game, they need to be placed in the root of your project folder. This is true for exported projects as well as when testing your game in the editor.

<a href='{{ site.url }}/assets/uploads/2014/05/matsnmodels.jpg'><img src='{{ site.url }}/assets/uploads/2014/05/matsnmodels-300x185.jpg' alt='Place the STK file in the root of your project.'></a>

## Loading packs

To load a pack file, we need to add it to the cache:

<pre><code>cache.addFile ( "MatsAndModels.stk", "file://" .. application.getPackDirectory ( ) .. "/MatsAndModels.stk" )</code></pre>

If the pack is successfully loaded, you will see the log message.

> Packfile : MatsAndModels.stk loaded from cache

If you don't see that message, then there is either a typo in your code or the STK file isn't in your root project folder.

## Referencing Resources

To use a model or resource that is in an STK file, you add the prefix of the STK name but without the .stk extension.

<pre><code>scene.createRuntimeObject ( hScene, box )</code></pre>

becomes

<pre><code>scene.createRuntimeObject ( hScene, MatsAndModels/box )</code></pre>

This is the same for sound, materials, xml, etc. Simply add the STK prefix with a slash and your resource(s) will be found.

NOTE. The models and resources should NOT be referenced in your game editor! This can be hard to get used to!

## Workflow Considerations

One complication with this way of organizing assets is that whenever a resource is updated, the STK file must also be updated. This can make for a terrible workflow when you want to do rapid prototyping. A simple work around is to first check if the resource is referenced by the project, if it isn't, check your list of known pack files. The idea is that you can prioritize referenced resources so that during rapid prototyping you can drag your resource(s) into the game editor so they'll override the contents of the pack files.

Here are two simple functions that do this.
            
<pre><code>--------------------------------------------------------------------------------
function ExternalSTK.getModel ( sModel )
--------------------------------------------------------------------------------
    
    --
    -- Checks to see if a model is referenced in the project and tries to fall
    -- back to known pack files if it isn't.
    --
    if application.isModelReferenced ( sModel ) then
        return sModel
    end
    
    --
    -- If you have multiple pack files, you may need to add more checks after this.
    --
    if application.isModelReferenced ( "MatsAndModels/" .. sModel ) then
        return "MatsAndModels/" .. sModel
    end
    
--------------------------------------------------------------------------------
end
--------------------------------------------------------------------------------</code></pre>
            
<pre><code>--------------------------------------------------------------------------------
function ExternalSTK.getResource ( sResource, kType )
--------------------------------------------------------------------------------
    
    --
    -- Checks to see if a resource is referenced in the project and tries to fall
    -- back to known pack files if it isn't.
    --
    if application.isResourceReferenced ( sResource, kType ) then
        return sResource
    end
    
    --
    -- If you have multiple pack files, you may need to add more checks after this.
    --
    if application.isResourceReferenced ( "MatsAndModels/" .. sResource, kType ) then
        return "MatsAndModels/" .. sResource
    end
    
--------------------------------------------------------------------------------
end
--------------------------------------------------------------------------------</code></pre>
            
Here's how I'd use these functions.

<pre><code>local hBox = scene.createRuntimeObject ( hScene, this.getModel ( "box" ) ) 
shape.setMeshMaterial ( hBox, this.getResource ( "red", application.kResourceTypeMaterial ) )</code></pre>

## Packaging Considerations

As of ShiVa 1.9.2, there are some considerations you should be aware of when packaging a game that has multiple STK files.

### Windows

On Windows, when a game is launched, the STK to load is chosen based on alphabetical order. You will need to:

1.  Modify the name of the STK file so that it is first in alphabetical order (**game.stk** becomes **agame.stk).**

**Mac**

On Mac, when a game is launched, the STK to load is chosen based on alphabetical order. You will need to:

1.  Modify the name of the STK file so that it is first in alphabetical order (**game.stk** becomes **agame.stk). **For a Mac app, the STK files are in YourGame.app/Contents/Resources/

**Linux**

Linux also has some STK load order issues but the best solution here is to make a bash file that deliberately calls the STK file to load. When you export a linux build, you get an executable baring the same name as your STK file. For **YourGame.stk** you will get an executable called **YourGame**. So the bash file would be:

<pre><code>#!/bin/bash
./YourGame YourGame.stk
</code></pre>
            
## Conclusion

The obvious application for STK files is that they can help deliver incremental updates to players in small manageable pieces. But there are other applications as well, like allowing players to mod your game by simply replacing your STK textures pack! Or customizing aspects of your game by loading different packs at runtime!

If you've used STK files in unique ways that aren't mentioned here, please share in the comments!

 [1]: https://github.com/error454/ShiVa-Proof-Of-Concept/tree/master/stkLoader

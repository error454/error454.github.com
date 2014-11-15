---
title: Barebones SDL For WebOS PDK
author: error454
layout: post
permalink: /2010/08/16/barebones-sdl-for-webos-pdk/
categories:
  - PDK
  - WebOS
tags:
  - palm
  - pdk
  - pixi
  - pre
  - sdl
  - webos
---
This article covers creating barebones SDL apps for the WebOS PDK.  The following topics are covered:

*   How to write a minimalist SDL app
*   How to set screen resolution based on device
<!--more-->


Ok, let's dust off those C skills and get to work.  First, of course, you need to setup your environment.  The Palm folks did such a good job explaining <a href="http://developer.palm.com/index.php?option=com_content&view=article&id=1974&Itemid=336" target="_blank">how to setup your development environment</a> that I would be silly to try any better.  This tutorial is written from a windows-centric perspective, I love my Mac friends too, I just don't have the privilege of owning one!

After you've installed the PDK, configured your environment and all that, it's time to start a project.  I find it easiest to copy the palm included simple sample project found at C:Program Files (x86)PalmPDKsharesamplecodesimple into a new directory.  When you load up the sample project you might be shocked to see how much code there is, don't panic, pretty much all this code is for OpenGL stuff. 

## A Minimalist SDL/PDK App

I'm all about keeping things simple, so I chopped down the the sample app to the barest essentials.  Here is my barebones version of hello world.

<pre>#include stdio.h
#include SDL.h

int main(int argc, char** argv)
{
    /* Even though we aren't using SDL for graphics
	 *  the PDK requires that Video be initialized
	 */
    SDL_Init(SDL_INIT_VIDEO);

	/* printf is deprecated, stdout is buffered
	 *  -use stderr so we can see output
	 */
    fprintf(stderr, Hello worldn);

    // Cleanup
    SDL_Quit();

    return 0;
}

</pre>

Palm includes 3 scripts in the sample that help you build the plugin executable and upload it to the device.  If you've renamed your project then you'll need to modify the highlighted references below so that sample and Sample.cpp point to the correct place location.

As you may have guessed, this app prints output to the console.  The only way to see this output is to ssh into your device and execute the app from there.  Not very practical for an app that runs on your phone, but now we have a barebones base to build on.

buildit.cmd

<pre>@echo off
@rem Set the device you want to build for to 1
set PRE=1
set PIXI=0
set DEBUG=0

@rem List your source files here
set SRC=Sample.cpp

@rem List the libraries needed
set LIBS=-lSDL -lGLESv2

@rem Name your output executable
set OUTFILE=sample
...
</pre>

runit.cmd

<pre>@echo off
call buildit.cmd
call uploadit.cmd
plink -P 10022 root@localhost -pw  /media/internal/sample
</pre>

uploadit.cmd

<pre>@echo off

plink -P 10022 root@localhost -pw  killall -9 gdbserver sample
pscp -scp -P 10022 -pw  sample root@localhost:/media/internal
</pre>

## Windows Libraries

Just a quick note, if you are testing the following code on your windows machine, you will start getting linker errors once we start using the PDL functions.  The errors you might see are unresolved symbols for \_WSACleanup \_gethostbyname \_gethostname and \_WSAStartup.  The solution here is to include a dependency for ws2_32.lib in your project linker dependencies.

[<img class="alignnone size-full wp-image-349" title="linker options" src="{{ site.url }}/assets/uploads/2010/08/linker.png" alt="" width="759" height="531" />][1]

Alternatively, you could add the following pragma to the code.

<pre>#pragma comment(lib, ws2_32.lib)
</pre>

## Setting Screen Resolution

Is the user running a Pre, a Pixi or maybe even a RoadRunner?  Let's find out and set the screen resolution appropriately.  Because I'm not really interested in printing to the console from here on out, I am going to remove the <stdio.h> include file along with the fprintf call.  There are 3 new data types in the following code:

*   <a href="http://www.libsdl.org/docs/html/sdlsurface.html" target="_blank">SDL_Surface</a> represents an area of memory that can be drawn to.
*   <a href="http://developer.palm.com/index.php?option=com_content&view=article&id=2067&Itemid=341#PDL_ScreenMetrics" target="_blank">PDL_ScreenMetrics</a> is a struct that holds information about the device's screen (look at the return types for this call, we will use them below)
*   <a href="http://developer.palm.com/index.php?option=com_content&view=article&id=2067&Itemid=422#PDL_Err" target="_blank">PDL_Err</a> is a struct that holds failure/success information for PDL calls

Bookmark the <a href="http://developer.palm.com/index.php?option=com_content&view=article&id=1990&Itemid=340" target="_blank">PDL libraries</a>, you'll reference them frequently.

<pre>#include SDL.h
#include PDL.h

SDL_Surface* screen;
PDL_ScreenMetrics screenMetrics;

int main(int argc, char** argv)
{
	//Initialize SDL
    SDL_Init(SDL_INIT_VIDEO);

    //Initialize the PDL library, this gives us access to the Plug-in API's
    PDL_Init(0);

    //Use a PDL call to collect screen metrics
	PDL_Err err = PDL_GetScreenMetrics(screenMetrics);

	//If the screen metric call failed, bail out
	if(err == PDL_INVALIDINPUT)
		return -1;

	//Fall back to 320x480 if not Pre or Pixi
	if(err == PDL_EOTHER)
	    screen = SDL_SetVideoMode(320, 480, 0, SDL_SWSURFACE);
	//Else if the device IS a pre/pixi, set the resolution based on the screen metrics
	else
		screen = SDL_SetVideoMode(screenMetrics.horizontalPixels, screenMetrics.verticalPixels, 0, SDL_SWSURFACE);

	//Pause for 5 seconds before exiting
	SDL_Delay(5000);

    // Cleanup
    PDL_Quit();
    SDL_Quit();

    return 0;
}
</pre>

I believe this code is fairly self-documenting, but let me point out a few lines.  Line 16, if you haven't been faithful to your C for awhile you may have forgotten about  notation which references the address of the variable, remember to look at function parameters to see whether they take a pointer.

On line 24, the <a href="http://www.libsdl.org/docs/html/sdlsetvideomode.html" target="_blank">SDL_SetVideoMode</a> call takes 4 arguments (width, height, bits per pixel (bpp), flags).  In our case, we are setting bpp to 0, this causes the bpp to be set to whatever the current bpp of the screen is.  We are also using the flag SDL_SWSURFACE.  Often times in the SDL world you will see people calling SDL_DOUBLEBUF | SDL_HWSURFACE to enable double buffering and write directly to video memory.  This <a href="https://developer.palm.com/distribution/viewtopic.php?f=70&t=5948" target="_blank">doesn't </a><a href="https://developer.palm.com/distribution/viewtopic.php?f=70&t=5948" target="_blank">appear to be implemented</a> in the PDK, so if you aren't using SDL_OPENGL here then you have to use SDL_SWSURFACE.

 [1]: {{ site.url }}/assets/uploads/2010/08/linker.png
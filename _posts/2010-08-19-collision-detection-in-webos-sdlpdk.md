---
title: Collision Detection for WebOS SDL/PDK
author: error454
layout: post
permalink: /2010/08/19/collision-detection-in-webos-sdlpdk/
categories:
  - PDK
  - WebOS
tags:
  - collision detection
  - pdk
  - sdl
  - sdl_blitsurface
  - sdl_fillrect
  - webos
---
## Collision Detection

<a href=''><img src='{{ site.url }}/assets/uploads/2010/08/yoda-shotgun.jpg' alt=''></a>

> An SDL padawan you are.
> 
> Drawn images to the screen using SDL_BlitSurface you have.
> 
> How to detect button presses you know not!
> 
> mMMmm, much to learn you still have!
<!--more-->
Collision detection isn't just for video games!  Imagine, your finger colliding with an image and then something epic happening.  This post is going to explore a simple way to detect when things collide.



What I will cover in this post is a very simple form of collision detection.  Quite simply, we will look at an X,Y coordinate for a tap event and then figure out if it fits inside a specified rectangle.  Before we begin, I should mention something that is very important about SDL_Rect.  The diligent student may notice that I never set the width/height of a rectangle in the below code.  Well, it just happens that when you call SDL_BlitSurface and specify a destination rectangle, the destination rectangle is modified to contain the width/height information of the bitmap you are blitting.  Keep this in mind. 

Is this lazy coding?  Should we manually set rectangle dimensions so that we don't have to worry about whether they've been blit'd?  Please comment if you have an opinion.  Now to the code.

<pre>#include SDL.h
#include PDL.h
#include SDL_image.h

//Surfaces
SDL_Surface* screen;
SDL_Surface* moon;
SDL_Surface* ship;
SDL_Surface* explosion;

//Rectangles
SDL_Rect moonLocation;
SDL_Rect shipLocation;

//Misc
PDL_ScreenMetrics screenMetrics;
SDL_Event sdlEvent;

bool isPointInRect(SDL_Rect r, Uint16 x, Uint16 y){
	return (r.x = x)
		 (x = r.x + r.w)
		 (r.y = y)
		 (y = r.y + r.h);
}

int main(int argc, char** argv)
{
	//Init
    SDL_Init(SDL_INIT_VIDEO);
    PDL_Init(0);

	//Set screen resolution and check for errors
	PDL_Err err = PDL_GetScreenMetrics(screenMetrics);
	if(err == PDL_INVALIDINPUT)
		return -1;
	if(err == PDL_EOTHER)
	    screen = SDL_SetVideoMode(320, 480, 0, SDL_SWSURFACE);
	else
		screen = SDL_SetVideoMode(screenMetrics.horizontalPixels, screenMetrics.verticalPixels, 0, SDL_SWSURFACE);

	//Load the images
	moon = IMG_Load(moon.png);
	ship = IMG_Load(ship.png);
	explosion = IMG_Load(explosion.png);
	if(moon == NULL || ship == NULL || explosion == NULL)
		return -2;
	moonLocation.x = screen-w/2 - moon-w/2;
	moonLocation.y = screen-h/2 - moon-h/2;

	//Draw the moon to the screen
	SDL_BlitSurface(moon, NULL, screen, moonLocation);
	SDL_Flip(screen);

	//The Event processing loop
	do {
		//Loop through all of the current events
		while(SDL_PollEvent(sdlEvent)){

			//Tap event
			if(sdlEvent.type == SDL_MOUSEBUTTONDOWN){
				//Move the ship to the clicked location
				shipLocation.x = sdlEvent.button.x - ship-w/2;
				shipLocation.y = sdlEvent.button.y - ship-h/2;

				//Clear the entire screen
				SDL_FillRect(screen, NULL, SDL_MapRGB(screen-format, 0, 0, 0));

				//Draw the moon
				SDL_BlitSurface(moon, NULL, screen, moonLocation);

				//If the location is inside the moon, draw an explosion
				if(isPointInRect(moonLocation, sdlEvent.button.x, sdlEvent.button.y))
					SDL_BlitSurface(explosion, NULL, screen, shipLocation);
				else //Draw the ship
					SDL_BlitSurface(ship, NULL, screen, shipLocation);

				//Draw the scene
				SDL_Flip(screen);
			}
			//Keyboard Event
			else if(sdlEvent.type == SDL_KEYDOWN){

				//Read the actual key that is down
				switch(sdlEvent.key.keysym.sym) {

					//Handle the escape key by exiting the program
					case SDLK_ESCAPE:
						sdlEvent.type = SDL_QUIT;
						break;

					default: break;
				}
			}
		}

		//Don't be a cpu hog
		SDL_Delay(1);
	} while (sdlEvent.type != SDL_QUIT);

    PDL_Quit();
    SDL_Quit();

    return 0;
}
</pre>

The majority of this code has been covered in previous WebOS PDK articles, please have a look under the PDK Category on the right.

## Collision Function

Let us begin with lines 19-24.  This function returns true if a point is contained inside a rectangle.  The function takes 3 parameters, an SDL_Rect along with an x,y coordinate.

<pre>bool isPointInRect(SDL_Rect r, Uint16 x, Uint16 y){
	return (r.x = x)
		 (x = r.x + r.w)
		 (r.y = y)
		 (y = r.y + r.h);
}
</pre>

The condition for a point to be inside a box is quite easy to describe and can be done using 2 inequalities.  As an aid, it may help to see a visual representation of this.

<a href=''><img src='{{ site.url }}/assets/uploads/2010/08/collision-square.png' alt=''></a>

Ok, just look at the rectangle for a moment.  Notice what the minimums and maximums for X and Y are:

X Min: x  
X Max: x+w  
Y Min: y  
Y Max: y+h

Given a point P that has component values for x and y, we can determine if the point is in the rectangle by comparing it to the min and max values of x and y.  First, the x component:

<strong>x <= P.x <= x+w </strong>

Then the y component:

**y <= P.y <= y+h**

If both statements are true then we are inside the rectangle.  The code in lines 20-23 just breaks up the above 2 inequalities into 4 inequalities, left side first, then right side.  It's quite simple once you can visualize the min/max of the rectangle.

## Blowing Things Up

62-63 set the location of the ship based on where the user taps.  The ship is centered on the tap.  Note that we are using shipLocation for both the location of the ship and the explosion.

<pre>shipLocation.x = sdlEvent.button.x - ship-w/2;
shipLocation.y = sdlEvent.button.y - ship-h/2;
</pre>

What we do next is clear the screen (66). Note that this call to SDL_FillRect can be troublesome if you attempt to take shortcuts like I did. You see, I figured that I would save some time by throwing in 0000000 as the color value. This worked fine in Visual Studio but resulted in a permanent black screen on the device. You can read through <a href="https://developer.palm.com/distribution/viewtopic.php?f=70&t=8474" target="_blank">my misadventures</a> if you are curious.  The important lesson is to use SDL_MapRGB to get the resulting Uint32 for the color.

<pre>SDL_FillRect(screen, NULL, SDL_MapRGB(screen-format, 0, 0, 0));
</pre>

72-75 is where we call the collision detection function and draw either an explosion or a ship, based on the results.

<pre>//If the location is inside the moon, draw an explosion
if(isPointInRect(moonLocation, sdlEvent.button.x, sdlEvent.button.y))
	SDL_BlitSurface(explosion, NULL, screen, shipLocation);
else //Draw the ship
	SDL_BlitSurface(ship, NULL, screen, shipLocation);
</pre>

That's it, here is a short video proving that it works!  


Credits:  
Explosion png was found at <a href="http://depts.washington.edu/cmmr/Research/XNA_Games/2008.5.R.1/Assignments/500_ClassHierarchy/XNA_Specifics/XNA_ImplementationGuide.html" target="_blank">Bothel University of Washington</a>.  
Moon png is from Nasa gallery.  
Ship png created by myself.
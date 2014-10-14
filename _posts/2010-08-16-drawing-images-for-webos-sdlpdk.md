---
title: Drawing Images for WebOS SDL/PDK
author: error454
layout: post
permalink: /2010/08/16/drawing-images-for-webos-sdlpdk/
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
## Drawing Images

Half of making a game is drawing the images to the screen, it's not too late to enter into the PDK contest folks!  Thankfully, drawing an image is simple with the functions provided by SDL_image.h.

<a href=''><img src='{{ site.url }}/assets/uploads/2010/08/sdlcap.jpg' alt=''></a>



Coincidently, you will need to add SDL_image.lib to your dependencies (or a pragma) for this to run on the desktop.  Before drawing an image, we first need to load it.  All images in SDL are of type SDL_Surface, this should look familiar because our screen object is also a surface, it just happens to be the surface that we draw everything else on to.

Did you ever play with <a href="http://www.thefeltsource.com/" target="_blank">felts</a> as a kid?  You know, where you had a big felt board that you stuck smaller felts to to make various scenes and stories?  Annoyingly, the small cut-out felts always faced the same direction, a fact that probably made budding young directors nervous.  So, surfaces are like felts, the screen is a giant SDL\_Surface upon which we place (blit) smaller SDL\_Surface objects.

Here is how to draw an image.  If a section of code is not commented below, it means that I covered it <a href="http://mobilecoder.wordpress.com/2010/08/16/barebones-sdl-for-webos-pdk/" target="_blank">earlier</a>.

<pre>#include SDL.h
#include PDL.h
#include SDL_image.h

SDL_Surface* screen;
SDL_Surface* image;
SDL_Rect imageLocation;
PDL_ScreenMetrics screenMetrics;
SDL_Event sdlEvent;

int main(int argc, char** argv)
{
    SDL_Init(SDL_INIT_VIDEO);
    PDL_Init(0);

	PDL_Err err = PDL_GetScreenMetrics(screenMetrics);
	if(err == PDL_INVALIDINPUT)
		return -1;

	if(err == PDL_EOTHER)
	    screen = SDL_SetVideoMode(320, 480, 0, SDL_SWSURFACE);
	else
		screen = SDL_SetVideoMode(screenMetrics.horizontalPixels, screenMetrics.verticalPixels, 0, SDL_SWSURFACE);

	//Load the image
	image = IMG_Load(moon.png);
	if(image == NULL)
		return -2;
	imageLocation.x = screen-w/2 - image-w/2;
	imageLocation.y = screen-h/2 - image-h/2;

	//Blit the image to the screen
	SDL_BlitSurface(image, NULL, screen, imageLocation);

	//Draw the screen
	SDL_Flip(screen);

	SDL_Delay(10000);

    PDL_Quit();
    SDL_Quit();

    return 0;
}
</pre>

First we have 2 definitions, an SDL_Surface pointer for our image and an <a href="http://www.libsdl.org/tmp/SDL-1.3-docs/structSDL__Rect.html" target="_blank">SDL_Rect</a> to hold the coordinates where the image will be displayed.  Line 26 is where we actually load the image using the IMG_Load function provided by SDL_image.h, the only parameter that this function takes is the file location, you might want to check the <a href="http://www.libsdl.org/projects/docs/SDL_image/SDL_image_9.html#SEC9" target="_blank">supported image formats</a>.  Make sure to put your image in your project directory so it can be found.  After checking to make sure that the image loaded successfully, we set the position of the image on the screen. 

<a href=''><img src='{{ site.url }}/assets/uploads/2010/08/box-coordinates.png' alt=''></a>

I've you never seen screen coordinates calculated, I'll explain the simple formula in lines 29/30.  Bitmaps are drawn based on the coordinates of a rectangle, the position of the rectangle is specified by a single vertex, which just happens to be the top-left vertex.

So you can see that the formula (screen width / 2) would put you directly in the center of the horizontal.  But if you draw the image at this coordinate, the top-left vertex of the image would be placed dead center, which isn't what we want.  So we need to offset the center by (image width / 2) which will properly center the image horizontally.  The same thing is done vertically.

Line 33 is where we blit the image onto the screen using <a href="http://www.libsdl.org/docs/html/sdlblitsurface.html" target="_blank">SDL_BlitSurface</a>.  Blit or Bit Blit is a fun word/phrase meaning that we are taking multiple bitmaps and turning them into a single bitmap.  This command literally copies our image onto the screen SDL_Surface at imageLocation.  The 2nd parameter is the source rectangle, setting it to NULL copies the entire source image.

## Updating the Screen

Finally on line 36 we display the screen object using <a href="http://sdl.beuc.net/sdl.wiki/SDL_Flip" target="_blank">SDL_Flip</a>.  Two things to note here, first, you only call SDL_Flip on a surface that has been allocated with SDL_SetVideoMode.  Finally, in the absence of double buffering, SDL_Flip is functionally equivalent to <a href="http://sdl.beuc.net/sdl.wiki/SDL_UpdateRect" target="_blank">SDL_UpdateRec</a>.  I prefer SDL_Flip because it only takes 1 parameter.

In SDL, you draw the entire scene starting at the background and working to the foreground.  So say you have a starfield background, a planet and a spaceship and you draw them in that same order.  Anytime the spaceship moves, you have to redraw the background, the planet and the spaceship, not just the spaceship.  In a game, this means you redraw every bitmap every frame!

There are basically two things you can do to optimize this.  The first is to reduce the number of blits.  For instance if you had a HUD that contained 20 separate bitmaps that don't change every single frame, it would be more efficient to blit the 20 bitmaps to a new surface, update this surface only when it changes and then blit this master surface to the screen.  Second, there is an SDL function that allows you to <a href="http://www.libsdl.org/cgi/docwiki.cgi/SDL_SetClipRect" target="_blank">set a clipping rectangle</a> on a surface to prevent certain areas of the surface from being written to.  Depending on the type of app you are writing, some creative clipping rectangles could save you from redrawing.
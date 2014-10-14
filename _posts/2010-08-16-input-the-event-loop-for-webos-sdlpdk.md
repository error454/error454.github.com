---
title: 'Input  The Event Loop for WebOS SDL/PDK'
author: error454
layout: post
permalink: /2010/08/16/input-the-event-loop-for-webos-sdlpdk/
categories:
  - PDK
  - WebOS
tags:
  - event
  - input
  - palm
  - pdk
  - pre
  - sdl
  - webos
---
## Reading Keyboard & Touch Events

SDL provides everything we need to read keyboard input and screen taps.  There is only 1 new data type that hasn't been covered previously, <a href="http://sdl.beuc.net/sdl.wiki/SDL_Event" target="_blank">SDL_Event</a>, the SDL wiki says it best:

> The SDL\_Event union is the core to all event handling in SDL; it's probably the most important structure after SDL\_Surface.

What we are about to do here is lay down the skeleton for event processing, the loop that keeps the app running, responds to input and displays output.  If you have an engineering background in feedback and control systems, we are about to define our transfer functions.  SDL_Event is a key player here since it allows us to get keyboard and tap events.



If you see code that isn't commented below, it is because I covered it in the <a href="http://mobilecoder.wordpress.com/2010/08/16/barebones-sdl-for-webos-pdk/" target="_blank">barebones article</a> or in <a href="http://mobilecoder.wordpress.com/2010/08/17/drawing-images-for-webos-sdlpdk/" target="_blank">Drawing Images</a>.

<pre>#include SDL.h
#include PDL.h

SDL_Surface* screen;
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

	//The Event processing loop
	do {
		//Loop through all of the current events
		while(SDL_PollEvent(sdlEvent)){

			//Tap event
			if(sdlEvent.type == SDL_MOUSEBUTTONDOWN){
				//Do something with mouse input
				//sdlEvent.button.x, sdlEvent.button.y contain x,y of tap
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

The first thing we do is setup the do/while loop.  The exit condition is for sdlEvent.type to be SDL_QUIT, this is a condition we will set manually when a user hits the escape key.  The inner while loop calls <a href="http://www.libsdl.org/docs/html/sdlpollevent.html" target="_blank">SDL_PollEvent</a> and passes in the sdlEvent variable created in line 6.  The sdlEvent is populated with the current event and we can then make decisions based on the event.  We handle two event types in this example.

1.  Tap events  We don't do anything here but we will soon
2.  Keyboard events  We detect if there are any keys down (SDL_KEYDOWN), if so then we read the actual key being pressed.  If that key is the escape key, we set sdlEvent.type to SDL_QUIT, our exit condition.

The last thing to note is line 49.  This loop will run continuously until the user exits, this means that when your app is completely idle it will still be using a healthy amount of CPU.  The solution is to put a small delay in the processing loop.  If you google around you will find a couple threads on this topic, along with debate between SDL_Delay(0) and SDL_Delay(1) as a minimum.  Specifying a 0 millisecond delay works on some systems because it ends up relinquishing a single timeslice, I typically use 1 as a minimum just to be safe.

When you begin polishing your app to be a good WebOS citizen, you will want to revisit SDL_Delay in your event loop and look to adjust the amount of delay based on whether your app is in the foreground or background.
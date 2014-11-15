---
title: Geeking Out with Philips Hue
author: error454
layout: post
permalink: /2014/11/02/geeking-out-with-philips-hue/
categories:
  - geek
tags:
  - hue
  - philips
  - python
  - node
---

I've been wanting to jump into smart lighting for a few years now. I finally decided that the Philips Hue system had enough of my desired features to warrant a starter kit. What follows is how I've been using the kit in various locations around my house.

#The Theater#
I use an OUYA to power my XBMC experience in my theater room. My primary desire in this room is lighting that automatically turns on and off based on whether content is playing. So if I pause a movie and need to get up, the lights go on, if I unpause then the lights go off. You can see a video of this below.

<iframe width="420" height="315" src="//www.youtube.com/embed/JojJ61P0Td0" frameborder="0" allowfullscreen></iframe>

<!--more-->
This solution is working at about 95%, there are still some edge case bugs to fix where sometimes I lose connection with XBMC. I'm leveraging a few open source projects to accomplish this:

* [phue](https://github.com/studioimaginaire/phue) is a python library for Philips Hue
* [node-xbmc](https://github.com/moul/node-xbmc) is an XBMC nodejs controller

##How it works##
All of this runs on my server which happens to be a linux box, but anything with python and node can do the job.

### Turning Lights On/Off ###
The core of the system is a set of simple python scripts for turning lights on and off. I leveraged the phue library to make my own helper library that turns most of my on/off scripts into about 3 lines of code. Here is an example from my helper library:

    #!/usr/bin/python
    from phue import Bridge
    
    b = Bridge('127.0.0.1')
    
    lights = b.get_light_objects('name')
    office = lights['Lamp']
    theater = lights['Theater']
    bedroom = lights['Bedroom']
    
    def On(lamp, hue, brightness):
        lamp.on = True
        lamp.effect = 'none'
        lamp.transitiontime = 0
        lamp.hue = hue
        lamp.brightness = brightness
        lamp.saturation = 254

Now to turn a light on, I can write a simple script:

    #!/usr/bin/python
    import zachhue
    
    zachhue.On(zachhue.theater, 46920, 200)

### Detecting XBMC Events ###
The 2nd piece to this is responding to XBMC pause/play/stop events. This is where the node-xbmc projects comes in. The node-xbmc project is a nodejs library that allows you to connect to your XBMC instance like a remote control. You can then register for events, for instance here is how I register for the pause and play events in javascript:

    xbmcApi.on('notification:pause', function(){
        console.log(Date.now() + ': onPause'); 
        mainLightsOn();
    });
    
    xbmcApi.on('notification:play', function(){
        console.log(Date.now() + ': onPlay');
        allLightsOff();
    });

**mainLightsOn()** is just a function which uses PythonShell to run one of my simple on/off scripts:

    function mainLightsOn(){
        PythonShell.run('lightson.py', function (err) {
            console.log(Date.now() + ': LIGHTS ON');
        });
    };

###Future Enhancements###
What I'd like in the end is to replace the rope light with accent lighting that shines on our movie posters. We also have rope lighting around the ceiling but are still waiting on our smart outlet to tie it into the system.

The hardest part of this project is remembering whether I'm writing python or javascript at any given time ;)

# The Office #
I was really impressed with the Philips Amibilight and some of the open source knockoffs. So I thought I'd try to copy this with what I had available.

The first step was to throw a lamp with one of the Hue bulbs behind my monitors. As you can see I have a corner desk, so it gives me some nice close planes to cast light onto.

<img src='{{ site.url }}/assets/uploads/2014/11/IMG_8355.jpg' alt='The lamp with hue bulb sites behind the monitors'>

Here is a shot that has my python script running, it is matching the color of the bulb to the average color of the content on my right monitor.

<img src='{{ site.url }}/assets/uploads/2014/11/IMG_8358.jpg' alt='The lamp with hue bulb sites behind the monitors'>

Here is a video showing it in action.

<iframe width="420" height="315" src="//www.youtube.com/embed/p8ScPT8N5Tg" frameborder="0" allowfullscreen></iframe>

Just in case anyone asks, some of the geek things in the shot are:

* Laser cut Zelda Box by [BurntPixels](https://www.etsy.com/shop/BurntPixels)
* Link pixel art by [8-Bit Babe](https://www.facebook.com/pages/8-BIT-BABE/379775135486551)

## How it Works ##
The workflow of this solution is simple.

1. Capture screenshot
2. Find average color in screenshot
3. Set Hue bulb to color found in #2

###Capturing the Image###
Since this is running on windows, I'm using the ImageGrab function from the PIL library. There are other cross-platform python routines for grabbing screenshots.

After capturing the image, I resize it to a width of 128 while matching the height to the aspect ratio of my monitor. This is to make the stats calculations have an easier time. To be honest, 128 was a totally arbitrary number that I picked only because 2^7 is always a good starting point.

    from PIL import ImageGrab
    from PIL import ImageStat

    width = 1920
    height = 1080

    # Grab the image
    im = ImageGrab.grab()

    # Resize to width of 128
    im = im.resize((128, int(128 / (width/height))))

###Analyzing the Image###
Now to find the average color of all the pixels captured in the screenshot. At first I was using ImageStat from PIL to do something like this:

    stats = ImageStat.Stat(im)
    # now stats.mean[0], stats.mean[1], stats.mean[2] are the r,g,b values

I noticed that the resulting average color would often be a color that wasn't even visible on the screen. I found an [old post from the python mailing list](https://mail.python.org/pipermail/python-list/2006-January/368904.html) and found that using the quantize method would be a good alternative since it would only return actual colors that exist in the image. So this is what I'm using now:

    qstats = im.quantize(1).convert("RGB").getpixel((0, 0))
    # now qstats[0], qstats[1], qstats[2] are the average r,g,b values

###Color Space Conversion###
Now that we have an average color in RGB, we need to convert from RGB to HSL. Here is the code snippet I'm using to do that. At first I tried using the **colorsys** library to do a conversion straight from RGB to HSB, but the hues didn't seem to match the Hue system very well. So I later found a stackoverflow post that linked over to the philips source with a good comment on how things work.

I converted the code found there into these 2 python functions that allow converting an rgb value to the CIE 1931 color space that the Hue uses.

{% gist 6b94c46d1f7512ffe5ee %}

Because the phue library has functions for setting CIE 1931 colors, we're done with this code:

    xy = zachhue.RGBtoXY(qstats[0], qstats[1], qstats[2])
    zachhue.OnXY(zachhue.office, xy, 255, 0.7)

Where OnXY is:

    def OnXY(lamp, xy, brightness, time):
        lamp.on = True
        lamp.effect = 'none'
        lamp.transitiontime = time
        lamp.xy = xy
        lamp.brightness = brightness

The one thing I'm still playing with is the brightness value. For now you can see above that I'm hardcoding it to 255. But I've also tried grabbing the actual brightness by converting the rgb -> hsv. The main issue with this is that during the day, the light isn't bright enough for me, hence the hardcoding of 255.

###Future Improvements###
Unfortunately, for my standard workflow, the average color of my screen is often pretty close to white :( Thanks every app in the world :/

The most obvious improvement is to replace the single bulb with 3 or 4 LED strips. Then you could capture select portions of the screen to drive the individual strips.

Burning CPU to analyze images continuously is not a great solution. It would be better for game devs to implement the Hue api directly into their game. I started hacking around with this in [Rage Runner](http://store.steampowered.com/app/279520) and it's pretty cool. Honestly the only thing scaring me away from a full integration is the thought of writing the necessary UI and logic to make a solid experience pairing the game to the Hue bridge :(

# The Bedroom #
I also have a Hue light in my bedroom. The only smarts on this one is a daily alarm that turns on the light at a certain time. My XBMC script also turns the bedroom light on between the hours of 9PM and 2AM if my XBMC box has just been turned off.
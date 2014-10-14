---
title: 'Lab Notes: Compiling ffmpeg for WebOS'
author: error454
layout: post
permalink: /2010/08/25/lab-notes-compiling-ffmpeg-for-webos/
categories:
  - PDK
  - WebOS
tags:
  - ffmpeg
  - ffplay
  - ffserver
  - hulu
  - pdk
  - rtsp
  - sdl_hwsurface
  - vlc
  - webos
---
<img class="size-full wp-image-429 alignleft" title="vlc" src="{{ site.url }}/assets/uploads/2010/08/vlc.png" alt="" width="102" height="102" />It is a dream of many to have VLC running on WebOS.  I thought that with the release of the PDK, I would start exploring this.  Since VLC uses many of the codecs provided by ffmpeg's libavcodec library, it made sense to start by compiling this library and verifying basic functionality.

In the end, I got ffplay to play a video on WebOS using libavcodec, the result was anti-climactic for large videos due to the lack of HW acceleration, one of the Palm PDK guys, unwiredBen provided <a href="http://developer.palm.com/distribution/viewtopic.php?sid=4ac3c9894ea8400e04ae978200205a51&lastaction=login&f=70&t=8505" target="_blank">information for further research</a>.  Overall, performance for smaller files is acceptable, and the ability to grab rtsp streams could lead to many applications.  The lab notes are below.



This isn't meant to be a tutorial but I will expound on things as  necessary.  I have a cross-compile configuration setup as<a href="http://www.webos-internals.org/wiki/WebOS_Internals_PDK" target="_blank"> outlined by the fine folks at WebOS Internals</a>.  If you configure Scratchbox 2 as outlined in the link, it makes these cross-compile tasks incredibly trivial.

## Getting the Source

<pre>git clone git://git.ffmpeg.org/ffmpeg/
cd ffmpeg
git clone git://git.ffmpeg.org/libswscale/
</pre>

## Config Options

If you're like me, the first thing I do when compiling source is to run ./configure help and copy/paste that output to notepad++/vi/gedit.  I then go through, line-by-line and delete options that aren't of interest to me.  At the end, I am left with a list of relevant config options.  Here is the list I came up with.  I use a **+** to designate that I want this option and a **-** to designate that I specifically want to make sure this option is disabled.  You may think that the **- **is redundant, it is not because it lets me later check default config options to make sure that the default for the option has not changed.

<pre>Configuration options:
-  --disable-static         do not build static libraries [no]
-  --enable-shared          build shared libraries [no]
-  --enable-gpl             allow use of GPL code, the resulting libs
                           and binaries will be under GPL [no]
-  --enable-nonfree         allow use of nonfree code, the resulting libs
                           and binaries will be unredistributable [no]
+  --disable-doc            do not build documentation
+  --enable-runtime-cpudetect detect cpu capabilities at runtime (bigger binary)
+  --enable-hardcoded-tables use hardcoded tables instead of runtime generation
</pre>

## Quick Source Analysis

ffmpeg provides source for a utility called ffplay.  I know that ffplay uses SDL and also know that the WebOS PDK doesn't support HWSURFACE.  I did a quick grep for SDL_HWSURFACE in ffplay.c and sure enough, there it was.

<pre>static int video_open(VideoState *is){
    int flags = SDL_HWSURFACE|SDL_ASYNCBLIT|SDL_HWACCEL;
    int w,h;
...
</pre>

<pre>case SDL_VIDEORESIZE:
	if (cur_stream) {
		screen = SDL_SetVideoMode(event.resize.w, event.resize.h, 0,
								  SDL_HWSURFACE|SDL_RESIZABLE|SDL_ASYNCBLIT|SDL_HWACCEL);
		screen_width = cur_stream-width = event.resize.w;
		screen_height= cur_stream-height= event.resize.h;
	}
...
</pre>

After changing both instances of SDL\_HWSURFACE to SDL\_SWSURFACE, I decided I felt lucky enough to compile.

## Compiling

From the scratchbox2 prompt:

./configure disable-doc enable-runtime-cpudetect enable-hardcoded-tables

<img src='' alt=''>

## **Testing**

I figured the easiest test would be to play the video that comes with the Pre.  I fired it up command-line style and SUCCESS!  But this video, an h264 at 320480 totally rocked the CPU on the Pre.  Top showed 99% CPU usage, I guess it's time to start looking into DSP coding on the TI OMAP3430.  I tried some lighter-weight flv files grabbed with youtube-dl and it plays those no problem, averaging about 25% CPU for a 320200 file.

## Next Steps

An app that leverages some simple web services along with ffserver/ffmpeg on the server side could be used to browse my audio/video library from the Pre.  I setup a quick ffserver/ffmpeg test where I re-encoded an avi using ffmpeg, pushed it out as an rtsp stream using ffserver and then slurped it down on the Pre with ffplay.  The test performed well, I pushed a 320240 video with a bitrate of 256 KB/s @20fps over Wifi.  Not surprisingly, my poor Atom processors were the bottleneck for HD video, only being able to re-encode at about 7fps.  Rtmpdump could be added to this mix for a HULU streaming solution (or any flash video for that matter).

There are 2 challenges with these solutions.

1.  How well will they work when you don't have Wifi?  The bottleneck when away from home is going to be the connection speed to your home server.
2.  These solutions are difficult to bring to the masses since a server is required.  Most folks with enough knowledge to do this stuff run Linux, so you'll need a Linux server if you want to follow along.

> Stream Hulu to your WebOS device, all you need is this app and a Linux server in your home!

That is a hard sell, but the world needs more Linux servers in the home <img src="http://www.error454.com/wp-includes/images/smilies/icon_wink.gif" alt=";)" class="wp-smiley" />
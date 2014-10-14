---
title: 'Note to self: Port mplayer to Touchpad'
author: error454
layout: post
permalink: /2011/09/25/note-to-self-port-mplayer-to-touchpad/
tagazine-media:
  - 'a:7:{s:7:"primary";s:0:"";s:6:"images";a:1:{s:74:"http://mobilecoder.files.wordpress.com/2011/08/applications-multimedia.png";a:6:{s:8:"file_url";s:74:"http://mobilecoder.files.wordpress.com/2011/08/applications-multimedia.png";s:5:"width";s:3:"150";s:6:"height";s:3:"150";s:4:"type";s:5:"image";s:4:"area";s:5:"22500";s:9:"file_path";s:0:"";}}s:6:"videos";a:0:{}s:11:"image_count";s:1:"1";s:6:"author";s:8:"11758919";s:7:"blog_id";s:8:"11929434";s:9:"mod_stamp";s:19:"2011-09-30 06:59:23";}'
categories:
  - Homebrew
  - WebOS
tags:
  - avi
  - draw_frame
  - draw_slice
  - ffmpeg
  - fragment shader
  - homebrew
  - mkv
  - mplayer
  - shader
  - stride
  - vlc
  - webos
  - yuv
---
<img class="alignleft size-full wp-image-931" title="TouchPlayer - the video player for total badasses" src="{{ site.url }}/assets/uploads/2011/08/applications-multimedia.png" alt="" width="120" height="120" />I have been working on porting mplayer over to the Touchpad.  When I work on projects, I write a note to myself at the end of the day to remember where I left off.  This is a copy of those notes.



**1:15 AM 11/6/2011**

I have just release version 1.0.6.  Talk about an incredibly painful release.  I got sidetracked with trying out mplayer2 vs mplayer and finally settled on sticking with mplayer.  It seems like the initial push for using mplayer2 was better matroska support, this was owed in large part to the ffmpeg-mt experimental branch.  At this point, the master FFmpeg branch has merged in the goodies from ffmpeg-mt so I didn't see much advantage in using mplayer2.  Add to that the odd play/pause issues I was having and it was an easy choice to move back to mplayer.

I got the freetype libraries in order which means no more bitmap fonts.  Now freetype fonts are available, so everything in /usr/share/fonts can be used.

I was hoping to find some more enhancements for MKV playback.  I keep meaning to do a full gprof build of libavcodec to see where cycles are being spent.  From the little benchmarking I've done, I believe that most of the CPU cycles are being spent on  audio.  At this point, you can play an MKV with audio disabled and subtitles on and it will play at full frame rate.  This wasn't possible before the NEON color space conversion.

Sure there are lots of UI enhancements that could be done, but I'm actually kind of tired of working on this project.  I've fully opened up the API and everything is open source to begin with, so if folks want more flash and features they can easily add them.  Until I encounter serious issues watching movies on my Touchpad, this project is hitting the back burner to simmer for a bit.

**6:00 AM 10/16/2011**
I have released version 1.0.4.  This actually came together quite quickly towards the end.  I was digging around the xbmc code and found a NEON accelerated YUV to RGB conversion function (I think this was pulled out of the chrome source), it makes MKV files play a lot smoother.  Still, not all of mine play perfectly.

I got the basic swiping/tapping controls in, still need to play with the sensitivity.

Now the big problem is the 1000s of command line variations.  How do I choose what to put into the GUI?  It seems like a task bound for defeat.  I've had the flu the last few days and in my feverish state I considered writing a web-based mplayer configuration file generator.  Must be the sickness talking.

Having issues with the commandline launcher, sometimes I can't launch, sometimes threads don't get cleaned up.  Pinged @jaycanuck to see if he has done any work on this, may need to look at how WOSI launches Xecutah.

**10:10 PM 10/14/2011**  
I now have subtitles and OSD working! The only oddity with the built-in matroska subs is that I have to hit j' to select the first subtitle index. This is annoying since there is only 1 available subtitle and none of the command line options seem to force this.

I spent a few minutes digging into the command and input stuff to determine how to call this manually. I didn't reach the end. Another thing I need to determine is how to package up the fonts, not just technically but also legally. I downloaded a packaged font pack for mplayer and extracted it to /media/internal and simply point mplayer to the font.desc (-font font-arial-cp1250/font-arial-18-cp1250/font.desc).

I'd like to implement behavior where a single tap pauses/resumes, swiping right/left scrubs, perhaps also a 2 finger swipe? Perhaps swiping up/down can enable/disable subs.

**9:01 PM 10/7/2011**  
Alone in a dark room, my keyboard illuminated only by the green characters of my VI session, I code deep into the night.  Tappity tap tap, click. . . I lean back and let out a sigh as I watch the compiler fill my screen with garbage.  Up enter, up up enter, tab up enter, I've repeated the process so many times now I almost do it without thinking.  The screen goes blank on my Touchpad and I prepare for another round of dissapointment as mplayer launches.

I AM SPARTACUS!!! the cry explodes from my Touchpad as bright vivid 720p erupts from my screen.   I try to shield my eyes from the sharp crisp image, it's so beautiful, so beautiful that I can't look away!  Hey wait, why the hell is it all choppy?

Theatrics aside, I'm not in a dark room coding late into the night.   I'm sitting at my evenly lit kitchen table laughing at my 2 year old niece, munching on leftovers and coding during the lulls in activity.   I've just gotten my YUV shader to work properly but it's no faster than doing the conversion in software, why?

After thinking and reading, one theory is that the whole pipeline is just too slow when using GLES2.   The problem is that you have to send 3 textures to the graphics card before conversion can be done.   So you get these 3 texture arrays (Y + U + V) which the CPU has to load into memory, then to use the shader for color conversion you have to copy these textures (again using CPU) from system memory to the GPU.

It seems that whatever time was saved by doing the color conversion in the GPU is more than made up for by the time it takes to load the textures into memory and copy them to the GPU.   Ideally we could short circuit this process by loading the textures into a Pixel Buffer Object, this is memory that is allocated directly on the GPU, then the copy would essentially be free, I mean the data is already there right? Unfortunately GLES2 does not support Pixel Buffer Objects :'(

How much data are we talking about here?  Let's see:

Y: 1280 x 720 bytes = 900 kB  
U: 640 x 360 bytes = 225 kB  
V: 640 x 360 bytes = 225 kB

So every frame is 900 + 225 + 225 = **1.3 MB**

At 30 frames per second this is 1.3 * 30 = **39 MB / second**

But hold on, is there really that big of a difference between copying 900 kB per frame vs 1.3 MB?  Really?  Is it really that much more work for the CPU?  It seems hard to believe.  I mean, any drawing method requires copying at minimum 900 kB to draw to the screen.  I really don't know, I suppose I would need to capture some performance metrics.

On the bright side, I now completely understand how different video formats are stored and how to write a fragment shader for color conversion. The question is what is next? How can I improve video performance knowing what I do?

Well first of all, I am going to stick with GLES2, but I'm going to try and avoid doing the color conversion with it.  The only real options for drawing to the screen are SDL or openGL, both drawing operations require that you copy what you want to draw from 1 place to another. So long as I'm only copying 1 RGB texture, whether I copy it to a shader or blit it to a surface doesn't matter. GLES2 still gives more flexability for doing post-processing, so that's what I'm going with.  I keep circling back on not being able to believe that things are as slow as they are, I really need to capture metrics.

What if I use NEON extensions to perform the color conversion, this would let me exploit SIMD without the overhead of the copy.  Assuming that the theory on the bottleneck is true.

**8:46 PM 10/6/2011**  
I posted a <a href="http://dsp.stackexchange.com/questions/365/yv12-to-rgb-what-is-wrong-with-my-algorithm" target="_blank">question</a> on dsp.stackexchange.com for help with my YUV conversion. It turns out that I was not taking the image stride into account. Microsoft has a great article on what image stride is (http://msdn.microsoft.com/en-us/library/windows/desktop/aa473780(v=vs.85).aspx) basically it is extra padding stored after the pixels. My guess is that it makes storing the frames in memory more efficient.

Inserting a print debug into the draw_slice function for a YV12 video, it printed the following continuously:

<pre>draw_slice: strideY: 1312 strideU: 656 strideV: 656 x: 0 y: 0 w: 1280 h: 720</pre>

So the textures I've been mapping to the fragment shader have been the wrong size all along.   Also, I was under the impression that draw\_slice() requested drawing of video frames in slices, like a grid a tiles that would be updated as needed.  Here I can clearly see that every draw call is drawing the entire frame.  Perhaps this is codec specific but the only difference between draw\_slice() and draw\_frame() from this perspective is that you get the stride in draw\_slice().

The current solution that I need to achieve is the cropping of the extra stride pixels in the video frame given to draw_slice().

More research, I found a GL parameter GL\_UNPACK\_ROW_LENGTH to specify the pixel format (stride/pack) of image data. Unfortunately this parameter is not available in GLES2 <img src="http://www.error454.com/wp-includes/images/smilies/icon_sad.gif" alt=":(" class="wp-smiley" /> 

More research, I see several folks setting an option in glTexParameter* called GL\_TEXTURE\_CROP\_RECT\_OES, unfortunately this seems to be some kind of extension that is not generally available in GLES2 <img src="http://www.error454.com/wp-includes/images/smilies/icon_sad.gif" alt=":(" class="wp-smiley" /> 

The two solutions that are common are not available, what now? It seems there are 2 paths:

1.  Copy the frame data to a memory location before uploading the textures. This lets me crop the image, getting rid of the extra stride pixels. Huge downside that this requires 3 memory copies (Y+U+V). This would be incredibly inefficient.
2.  Bind the stride data in the shader and perform the image crop in the shader. The downside here is that I have no clue how to do this, but as it seems it is the better solution, I will focus research effort down this path.

The fragment shader provides a built-in variable gl_FragCoord that is supposed to give you the window coordinates of the current fragment. I'm hoping I can use this along with the discard command to skip the color calculation for fragments that exist in the stride.   What I'm not sure of is whether the shader factors texture clamping into the calculation . . . i.e. when using edge clamping for my texture, will the fragment coordinate of an area in the stride be > 1.0 or == 1.0?  Wow, no clue!   Looks like I need to figure out how to print debug data from the fragment shader.

**8:52 PM 10/4/2011**  
More work on a shader for YUV420p, something is clearly wrong here.  I think it must have to do with the content coming in through draw_slice.  I've tried every YUV fragment shader I could find and finally settled on a modified version that follows some of the equations and discussion points on the fourcc website.  All of them essentially look like this, I need to post this on a stackexchange site and see if someone can identify what the decoding issue might be. . . Other than that, the framerate seems ok.  I've been hung up on this for quite some time, I may have to back-burner the YUV shader and get the other low-hanging fruit done like the OSD.  At least we'd have a fully functioning player with subtitles, it just wouldn't have accelerated colorspace conversion for h.264.

<a href='{{ site.url }}/assets/uploads/2011/09/sdl_2011-04-10_014619.png'><img src='{{ site.url }}/assets/uploads/2011/09/sdl_2011-04-10_014619.png?w=300' alt=''></a>

**3:46 PM 10/2/2011**  
I made an attempt to get the YUV shader going. At first nothing was being drawn to the screen with even a simple shader, I made some changes to the program loader so that it does a hard fail if the vertex or fragment shader fails to compile. Now that I know the shader is at least good in theory, I tried several shaders that I found out on the interwebs for YUV conversion. I can't get an intelligable picture using any of the implementations, I'm guess it is due to these conversion formulas expecting non-planar data.

**12:43 AM 9/29/2011**  
I finally had some time to put towards this. I was able to get in correct aspect ratio scaling for the texture coordinates so squares and circles now look like squares and circles. It could be worth revisiting once the OSD is enabled to allow for different scaling methods, right now the height is modified to achieve the correct aspect ratio (this works since it's a 4:3 screen).

MKV playback is a little better but it still doesn't run at full framerate.  I've seen some discussion about doing the YUV conversion in a fragment shader, one side claims it's faster because the calculations are being done in the GPU while the other side claims it's slower because you have to copy 3 textures from memory.  If anyone knows a definitive answer please say so.  Right now the YUV conversion is not being done in a fragment shader although I had hoped to move that direction.

I'm getting a segfault when exiting mplayer and this oddly makes mplayer stick around in the process list.  Also I need to add in the event loop check to pause video on card minimize but I might just wait for the OSD since there's no point in pausing if you can't hit play again.

**9:03 AM 9/25/2011**
I contacted Chomper who got back to me a few hours later and sent an updated version of vo_sdl.c.  This version uses libswscale to do the yuv conversion and also has some other optimizations.  He also said there were some other bugfixes to be done.

I compiled the new stuff from Chomper and MKV playback made a huge leap in performance.   It went from unplayable to near 24 fps with audio enabled.

At this point I am trying to get subtitles and OSD enabled but am having issues getting the configuration step to recognize libfreetype.

Not sure if I should compile this myself or try to point to the libs/headers from the preware cross compile.   I don't want any other libs in there to be accidentally pulled in over my optimized compiled versions.

For future note, I have been using the following configure settings:  
extra-libs=-lGLESv2 enable-neon enable-gl disable-dvdnav disable-dvdread disable-dvdread-internal disable-libdvdcss-internal disable-alsa enable-menu extra-cflags='-O3 -funroll-loops -marm -march=armv7-a -ftree-vectorize -mfpu=neon -mfloat-abi=softfp -mtune=cortex-a8 -I/opt/PalmPDK/include/ -I/opt/PalmPDK/include/GLES2 -L/opt/PalmPDK/device/lib/'

Also, I was doing some research on the best build options for performance on armv7 and stumbled on <a href="http://www.cnx-software.com/2011/04/22/compile-with-arm-thumb2-reduce-memory-footprint-and-improve-performance/" target="_blank">this</a> page.  I've been building ffmpeg and mplayer with the options that they say get the best coremark scores:  
-O3 -funroll-loops -marm -march=armv7-a -mtune=cortex-a8

**10:56 AM 9/23/2011**  
I love pushing code via freetether while riding public transit!  I have now integrated the WOSI/chomper openGL ES code into my mplayer fork. The code compiles but there are a couple things I need to revisit:

1.  The code currently hijacks the vo\_sdl.c output module. It would make more sense if we kept vo\_sdl.c the same and made a new vo\_gles.c module. This will require a few additions to video\_out.c (correct name?).
2.  The screen initialization code in the above code specifies a width, height, bit depth. I hard-coded the bit depth to 0 and think I should do the same for width/height (as per Palm recommendation, pretty sure this just autodetects dimensions of screen).
3.  The patch I hand applied turned all of the OSD stuff into no-ops. Once I have a chance to test the actual video performance, I need to revisit this and get the OSD stuff back in. Possibly I may need to look at the existing openGL module and convert the OSD contents to GL ES.

**1:03 AM 9/22/2011**  
I now have mplayer compiled for ARM. I am unable to get SDL video output working, I double-checked that SWSURFACE is being used. I can use fbdev output but this doesn't pull up in an app window and doesn't fill the whole screen.

I would really like to be able to use openGL as the output but mplayer does not support openGL ES. I would basically have to copy vo_gl.c and make an ES version.

The advantage of doing this in mplayer rather than ffplay is primarily being able to use the OSD and the numerous skins available for it (I think). I'm actually unsure if I can even use the OSD with all the GTK stuff.

I should take a look at the chomper patch for mplayer, it is old but might be enough to help me figure out how to get SDL working. Once this works, I can compile with the OSD stuff enabled and see whether I can at least get the OSD going.
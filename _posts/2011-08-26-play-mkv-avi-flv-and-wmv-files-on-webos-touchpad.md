---
title: Play mkv, avi, flv and wmv files on webOS Touchpad
author: error454
layout: post
permalink: /2011/08/26/play-mkv-avi-flv-and-wmv-files-on-webos-touchpad/
tagazine-media:
  - 'a:7:{s:7:"primary";s:0:"";s:6:"images";a:1:{s:74:"http://mobilecoder.files.wordpress.com/2011/08/applications-multimedia.png";a:6:{s:8:"file_url";s:74:"http://mobilecoder.files.wordpress.com/2011/08/applications-multimedia.png";s:5:"width";s:3:"150";s:6:"height";s:3:"150";s:4:"type";s:5:"image";s:4:"area";s:5:"22500";s:9:"file_path";s:0:"";}}s:6:"videos";a:0:{}s:11:"image_count";s:1:"1";s:6:"author";s:8:"11758919";s:7:"blog_id";s:8:"11929434";s:9:"mod_stamp";s:19:"2011-08-26 07:29:04";}'
categories:
  - Homebrew
  - WebOS
tags:
  - avi
  - ffmpeg
  - homebrew
  - mkv
  - mplayer
  - vlc
  - webos
---
<img class="alignleft size-full wp-image-931" title="TouchPlayer - the video player for total badasses" src="{{ site.url }}/assets/uploads/2011/08/applications-multimedia.png" alt="" width="120" height="120" />

This article is no longer maintained, please see [TouchPlayer Documentation][1].
<!--more-->
One thing the touchpad clearly lacks is the number of supported video formats available. Out of the box, the mp4 container with h.264 codec is pretty much it.  If all of your videos are in this format already, you can benefit from the hardware acceleration that the stock video player provides for this codec.

If you want to play any other format, you have a few options:

1.  Convert your existing media to mp4 h.264 with an app like [Handbrake][2]. Leave it running overnight to take advantage of spare CPU cycles.
2.  Run a media server like [Playon][3] that transcodes media on the fly.  You need a reasonably fast cpu to do this (atom powered servers do not apply) as you are converting and serving the video real-time.
3.  Get a new video player for your touchpad.

This article is going to explore the 3rd option in more detail.

<h1 class="more">
  Presenting TouchPlayer
</h1>

I have taken the FFmpeg project, thrown a crappy wrapper on it and compiled it for the Touchpad.  If you aren't familiar with FFmpeg, the library that comes from it (libavcodec) powers mplayer and vlc.  You can find my app in Preware, it is called TouchPlayer.

# TouchPlayer Limitations

**Update 10/16/2011**

1.0.4 is [now available][4].

**Update 10/14/2011**

First off, let me say that if you are looking to play MKV files, you should really check out KalemSoft Media Player.  I use it, it's fast, it's beautiful etc.

There are limitations with KalemSoft though, such as reading files from an NFS/CIFS/SSHFS mounted filesystem.  Also, no subtitles.

I have abandoned the ffplay version of this project in favor of mplayer.  Progress is steady, you can [read progress updates][5] if you'd like.  An update has not yet been pushed, the mplayer version is all or nothing, meaning if I released it as-is there would be no controls.

**Update 9/23/2011**

It's possible that I don't know what I'm talking about below.  KalemSoft media player can play most of my HD MKV files quite well.  The big difference here is that they seem to draw to the screen using openGL rather than SDL.  This may not be the only factor but I'm sure it is a large factor.

As ffplay is something of a dead-end, I am now working on replacing it with mplayer.  There are a couple reasons, first there is already an openGL ES patch for mplayer from either WOSI or Chomper (hard to track the attribution for the patch).  Also, mplayer has an OSD and a GUI with several skins available.

The project is building but I haven't had the time to test it on the Touchpad.  You can find the source on my github page if you are eager.

**Old Content**

TouchPlayer cannot play 720P mkv videos with AC3 audio at 30 fps.  <del>It never will unless the Touchpad gets more hardware accelerated codecs. </del>Who knows, maybe it will someday.

Keep in mind that every video played in TouchPlayer gets decoded by the CPU.  <del>Decoding 720p with multi-channel audio on the CPU is too big of a task for any tablet.  If your primary goal is to play unmodified mkv files then you may want to consider new goals.</del> Using NEON, it is possible to do, all on the CPU.

If you insist though, I have provided a switch to disable audio and increase thread count to take advantage of both CPU cores.  If you do have any level of success with playing hd videos, please let me know.

Also if you are an expert in the FFmpeg ecosystem and know of some parameters that might give a boost, please pass them along.

Source for the modified FFmpeg is here:  
<a href="https://github.com/error454/FFmpeg" target="_blank">https://github.com/error454/FFmpeg</a>

Source for TouchPlayer is here:  
<a href="https://github.com/error454/TouchPlayer" target="_blank">https://github.com/error454/TouchPlayer</a>

Precentral Forum on TouchPlayer:  
<a href="http://forums.precentral.net/webos-homebrew-apps/293654-touchplayer.html" target="_blank">http://forums.precentral.net/webos-homebrew-apps/293654-touchplayer.html</a>

Direct link:  
[http://t.co/gnKEkkt][6]

 [1]: http://mobilecoder.wordpress.com/2011/10/16/touchplayer-documentation/
 [2]: http://handbrake.fr/
 [3]: http://www.playon.tv/index.php
 [4]: http://error454.com/2011/10/16/touchplayer-documentation/
 [5]: http://error454.com/2011/09/25/note-to-self-port-mplayer-to-touchpad/
 [6]: http://t.co/gnKekkt
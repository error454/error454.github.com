---
title: 'WebOS Camera Patch: Advanced Camera Configuration'
author: error454
layout: post
permalink: /2011/07/10/webos-camera-patch-self-timer-and-burst-frames/
tagazine-media:
  - 'a:7:{s:7:"primary";s:90:"http://mobilecoder.files.wordpress.com/2011/07/camera_2011-10-07_200059-e1310353914929.png";s:6:"images";a:2:{s:75:"http://mobilecoder.files.wordpress.com/2011/07/camera_2011-10-07_200026.png";a:6:{s:8:"file_url";s:75:"http://mobilecoder.files.wordpress.com/2011/07/camera_2011-10-07_200026.png";s:5:"width";s:3:"320";s:6:"height";s:3:"480";s:4:"type";s:5:"image";s:4:"area";s:6:"153600";s:9:"file_path";s:0:"";}s:90:"http://mobilecoder.files.wordpress.com/2011/07/camera_2011-10-07_200059-e1310353914929.png";a:6:{s:8:"file_url";s:90:"http://mobilecoder.files.wordpress.com/2011/07/camera_2011-10-07_200059-e1310353914929.png";s:5:"width";s:3:"480";s:6:"height";s:3:"320";s:4:"type";s:5:"image";s:4:"area";s:6:"153600";s:9:"file_path";s:0:"";}}s:6:"videos";a:0:{}s:11:"image_count";s:1:"2";s:6:"author";s:8:"11758919";s:7:"blog_id";s:8:"11929434";s:9:"mod_stamp";s:19:"2011-07-11 03:12:33";}'
categories:
  - Homebrew
  - WebOS
tags:
  - burst
  - camera
  - patch
  - timer
  - webos
---
**UPDATE 2: **What was formerly the Self-Timer and Burst-Frame patch has been replaced.  To maintain compatibility with older camera patches, the new patch combines the following patches into a single Tweaks-supported patch:

*   Video camera flashlight
*   Capture with volume-key
*   Shutter sound
*   Self-Timer and Burst-Frames
<!--more-->
<del><strong>UPDATE: </strong>Did you come here wondering why the camera patch will not install?  Unfortunately, due to code ninjas, there are a few patches that this patch does not play well with  the video camera flashlight patch and a few naming patches.  I am looking into things and am beginning to form a solution that I hope will clean up the camera patch ecosystem.  Stay tuned.</del>

Please leave comments below or twitter me if you have issues or a feature request for this patch.

[<img class="aligncenter size-full wp-image-889" title="camera_2011-10-07_200026" src="{{ site.url }}/assets/uploads/2011/07/camera_2011-10-07_200026.png" alt="" width="320" height="480" />][1]<img class="aligncenter size-full wp-image-890" title="camera_2011-10-07_200059" src="{{ site.url }}/assets/uploads/2011/07/camera_2011-10-07_200059-e1310353914929.png" alt="" width="480" height="320" />

Usage Note:

When the camera app first starts, a service call is made to query all of the Tweaks settings.  This service call can take a second or two to return at worst.  As a result, you may notice that your configured settings do not take affect until 1-2 seconds after the camera app has initialized.

Changelist:

Version 1.0.0

*   Self-timer enable/disable
*   Self-timer custom time setting
*   Burst frames enable/disable
*   Burst frames custom frame count
*   Burst frame shutter release time.  Note that this value may not work well below 2 seconds if you are using flash.  Even if you aren't, I've had hiccups where the device sometimes misses a frame.
*   Shutter-sound enable/disable
*   Capture with volume keys enable/disable
*   Video camera flashlight enable/disable

 [1]: {{ site.url }}/assets/uploads/2011/07/camera_2011-10-07_200026.png
---
title: 'Lab Notes  Hacking file_storage.c'
author: error454
layout: post
permalink: /2011/03/25/lab-notes-hacking-file_storage-c/
categories:
  - Homebrew
  - WebOS
tags:
  - file_storage.c
  - hacking
  - kernel
  - mass storage
  - webos
---
## Progress Update

I made attempt 3 to use file\_storage.c as a means of presenting an ISO image on the Pre as a CD-ROM drive over USB.  I tried grabbing several of the more recent versions of file\_storage.c from git.kernel.org in an attempt to back-port them to the 2.6.24 kernel.

I found the following:

*   file_storage.c got CD-rom support in January of 2009:
*   <pre>vermagic:       2.6.32-28-server SMP mod_unload modversions
parm:           file:names of backing files or devices (array of charp)
parm:           ro:true to force read-only (array of bool)
parm:           luns:number of LUNs (uint)
parm:           removable:true to simulate removable media (bool)
parm:           stall:false to prevent bulk stalls (bool)
<strong>parm:           cdrom:true to emulate cdrom instead of disk (bool)</strong><strong>
</strong></pre>

*   At some point, part of the contents of file\_storage.c was moved to storage\_common.c.  From this point on, the backport seems more difficult.

## Compile Notes

I proceeded by using the January 2009 version, there was really no work needed to compile this in place of the old version.  I did need to make one change.  By default, if you load the resulting module (g\_file\_storage), it will attempt to use the same sysfs entries as f\_mass\_storage.c - */sys/devices/platform/musb_hdrc.0/gadget/gadget-lunX. * Attempting to insert the module is very disappointing when this occurs.  To work around this, I changed the sysfs entry name by modifying the following line  in fsg_bind() from

<pre>snprintf(curlun-dev.bus_id, BUS_ID_SIZE,
                                %s-lun%d, gadget-dev.bus_id, i);
</pre>

to

<pre>snprintf(curlun-dev.bus_id, BUS_ID_SIZE,
                                %s-cdlun%d, gadget-dev.bus_id, i);
</pre>

This enumerates the lun entries starting with **gadget-cdlun0**.

## Results

The results are not pleasing.  I tried 3 tests:

### 1. Presenting an Ubuntu ISO from the Pre to a Windows 7 box.

The OS detected the device as File-CD based storage gadget, took about 5 minutes to install the driver and then didn't see anything presented.  I noted that if I rmmod'd the module, I would see a CD-ROM drive pop-up in windows explorer temporarily  ultimately not helpful.

### 2. Presenting an Ubuntu ISO from the Pre to a computer on boot.

The BIOS post confirmed that there was a File-CD based storage gadget plugged in.  I attempted to boot off the device and got the familiar message that my boot media was invalid.

### 3. Presenting an Ubuntu ISO from the Pre to a linux box.

After plugging this in and watching /var/log/messages fly by, I saw that the system discovered the gadget.  It did take an unusual amount of time for the system to mount the device after discovering it (~20 second).  After the device was mounted, I had no problems accessing the contents of the ISO.

## Unknowns

*   Is the Rockhopper driver muddying the waters?  It has been loaded for all of these tests just because I didn't want to recompile my kernel to remove it.
*   Should I just wait this out until 3.0 and a newer kernel?
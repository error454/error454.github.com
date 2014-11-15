---
title: Using the Palm Pre as Xbox 360 Storage (the finale)
author: error454
excerpt: 'Palm Pre as Xbox 360 Mass Storage finally solved.  The solution also has a pleasant bi-product of enabling Android devices on Xbox 360 as well.'
layout: post
permalink: /2010/09/16/using-the-palm-pre-as-xbox-360-storage-the-finale/
categories:
  - Android
  - WebOS
tags:
  - android
  - droid
  - droid2
  - evo
  - mass storage
  - sd
  - storage
  - usb
  - webos
  - xbox
  - xbox 360
  - xbox360
---
<a href=''><img src='{{ site.url }}/assets/uploads/2010/09/booyah.png' alt=''></a>

15 days ago, I set out to determine why I couldn't use my Palm Pre on my Xbox 360.  We all start somewhere, but I started with absolutely no knowledge of the composite gadget framework or Palm's rockhopper composite driver.  In retrospect, I spent a lot of time asking silly questions and researching irrelevant things.  But once again I have been awarded for persistence, while picking up some great knowledge and tools for the future.
<!--more-->
For the last 2 days, I have relished the joy of discovery.  Truth be told, this problem was solved 2 years ago by one Felipe Balbi on the linux-usb mailing list.  The next logical question is why the problem exists on so many devices if it was fixed 2 years ago?  Am I really the first person to discover this problem and solution and actually care enough to try and get it fixed on Palm and Android devices?

Read on as I reveal all that I know. (Palm users can skip to the good stuff by grabbing the latest UberKernel release and plugging in to their Xbox 360)



## Setting the Stage

I am going to describe the process by which data is transfered from a host to a device using the BOT protocol.  Before doing so, I need to present two of the data structures that are used to send data and reply with status.  These data structures are defined in great detail in the <a href="http://www.usb.org/developers/devclass_docs/usbmassbulk_10.pdf" target="_blank">spec for the USB Mass Storage Class</a>.  The definitions here are pulled from Alan Stern's monolithic file storage gadget.

### CBW

<pre>/* Command Block Wrapper */
struct bulk_cb_wrap {
        __le32  Signature;              /* Contains 'USBC' */
        u32     Tag;                    /* Unique per command id */
        __le32  DataTransferLength;     /* Size of the data */
        u8      Flags;                  /* Direction in bit 7 */
        u8      Lun;                    /* LUN (normally 0) */
        u8      Length;                 /* Of the CDB, = MAX_COMMAND_SIZE */
        u8      CDB[16];                /* Command Data Block */
};
</pre>

### CSW

<pre>/* Command Status Wrapper */
struct bulk_cs_wrap {
        __le32  Signature;              /* Should = 'USBS' */
        u32     Tag;                    /* Same as original command */
        __le32  Residue;                /* Amount not transferred */
        u8      Status;                 /* See below */
};
</pre>

I have to point out something incredibly useful that is defined in the spec.

For CBW -

> <a href=''><img src='{{ site.url }}/assets/uploads/2010/09/booyah.png' alt=''></a>
> 
> <a href=''><img src='{{ site.url }}/assets/uploads/2010/09/booyah.png' alt=''></a>

For CSW -

> dCSWSignature:
> 
> Signature that helps identify this data packet as a CSW. The signature field shall contain the value 53425355h (little endian), indicating CSW.

How does this help?  For those of us running usbmon, you can easily identify CBW and CSW which will allow you to reverse engineer the data contained in the structures.  For instance, this string from usbmon:

<pre>ffff88008d4e0480 975285546 C Bi:2:040:1 0 13 = 55534253 01000000 00000000 00
ffff88008d4e0480 975286954 S Bo:2:040:1 -115 31 = 55534243 02000000 00000000 00000600 00000000 00000000 00000000 000000
</pre>

In the first line, 55534253 identifies a CSW (convert to little endian), whereas the second line 55534243 is a CBW!  When transferring data using the BOT protocol, the basics work like so, to keep things simple I'll refer to the host as the Xbox and the device as the Pre.

1.  The Xbox sends a CBW to the Pre (this arrives on the Bulk-Out EP)
2.  The Pre analyzes the CBW and attempts to satisfy the host request
3.  The Pre returns status to the Xbox regarding the results of the CBW using a CSW

<a href='http://www.usb.org/developers/devclass_docs/usbmassbulk_10.pdf'><img src='{{ site.url }}/assets/uploads/2010/09/bot-flow.png' alt=''></a>

All <a href="http://en.wikipedia.org/wiki/SCSI_command" target="_blank">SCSI commands</a> are wrapped by the CBW struct.  Regardless if we are reading/writing data or requesting device information, all of these commands arrive in a CBW payload.

## The Problem

The problem, <a href="http://article.gmane.org/gmane.linux.usb.general/10058" target="_blank">originally patched by Felipe Balbi</a> back in September of 2008 is best described by the author.

<pre>/* Special case workaround: There are plenty of buggy SCSI
 * implementations. Many have issues with cbw-Length
 * field passing a wrong command size. For those cases we
 * always try to work around the problem by using the length
 * sent by the host side provided it is at least as large
 * as the correct command length.
 * Examples of such cases would be MS-Windows, which issues
 * REQUEST SENSE with cbw-Length == 12 where it should
 * be 6, and xbox360 issuing INQUIRY, TEST UNIT READY and
 * REQUEST SENSE with cbw-Length == 10 where it should
 * be 6 as well.
 */
</pre>

So going back to steps 1-3 above, here is what happens:

The Pre receives the CBW, it looks at the length of the CDB and notes that it is wrong.  Because the correct logic doesn't exist to handle this particular size (10), the request is dropped on the floor.

## The Solution

The solution is quite simple, check the length of the command and fix it if need be.  A more generic fix that can handle varying faulty request lengths is what Felipe submitted.

## Why the Problem Still Exists

In 2008, Mike Lockwood from the Android team created f\_mass\_storage.c which was based on file_storage.c by Alan Stern.

In Sept 2008, a patch for file\_storage.c was submitted that fixes functionality with buggy SCSI implementations.  This patch was never carried over to the Android f\_mass_storage.c.

In 2009, Palm released the Palm Pre using the Android version of f\_mass\_storage.c as-is.

## The Final Solution

In the mainline kernel there is a new f\_mass\_storage.c written by Michal Nazarewicz, also based on file_storage.c.  This new driver includes the fix for the Xbox 360.  Android<a href="http://www.mail-archive.com/android-kernel@googlegroups.com/msg01931.html" target="_blank"> has abandoned their copy of f_mass_storage.c</a> in favor of the mainline version.  Unfortunately, since they just did this recently it means that Droid/DroidX/Droid2/EVO users will not be able to use their device with the Xbox 360 until Android 2.3 is released.

My recommendation to Palm is to adopt the mainline f\_mass\_storage.c.  Until then, I am using the following patch to allow my Palm Pre to act as mass storage for the Xbox 360. This patch has been<a href="http://git.webos-internals.org/?p=kernels/patches.git;a=blob;f=usb/f_mass_storage.patch;hb=HEAD" target="_blank"> submitted to the WebOS Internals team</a> and <s>will hopefully show up in a future UberKernel release</s> is in the latest UberKernel.

<pre>diff -BurN linux-2.6.24/drivers/usb/gadget/f_mass_storage.c linux-2.6.24/drivers/usb/gadget/f_mass_storage.c.fix
--- linux-2.6.24/drivers/usb/gadget/f_mass_storage.c    2010-09-16 16:37:58.651375877 -0700
+++ linux-2.6.24/drivers/usb/gadget/f_mass_storage.c.fix        2010-09-16 16:35:10.783868776 -0700
@@ -1980,11 +1980,24 @@
        /* Verify the length of the command itself */
        if (cmnd_size != fsg-cmnd_size) {
-               /* Special case workaround: MS-Windows issues REQUEST SENSE
-                * with cbw-Length == 12 (it should be 6). */
-               if (fsg-cmnd[0] == SC_REQUEST_SENSE  fsg-cmnd_size == 12)
-                       cmnd_size = fsg-cmnd_size;
-               else {
+               /* Special case workaround: There are plenty of buggy SCSI
+                * implementations. Many have issues with cbw-Length
+                * field passing a wrong command size. For those cases we
+                * always try to work around the problem by using the length
+                * sent by the host side provided it is at least as large
+                * as the correct command length.
+                * Examples of such cases would be MS-Windows, which issues
+                * REQUEST SENSE with cbw-Length == 12 where it should
+                * be 6, and xbox360 issuing INQUIRY, TEST UNIT READY and
+                * REQUEST SENSE with cbw-Length == 10 where it should
+                * be 6 as well.
+                */
+               if (cmnd_size = fsg-cmnd_size) {
+                       DBG(fsg, %s is buggy! Expected length %d 
+                                       but we got %dn, name,
+                                       cmnd_size, fsg-cmnd_size);
+                       cmnd_size = fsg-cmnd_size;
+               } else {
                        fsg-phase_error = 1;
                        return -EINVAL;
                }
</pre>
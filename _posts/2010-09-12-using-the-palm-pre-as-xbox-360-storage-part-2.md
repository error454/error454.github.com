---
title: Using the Palm Pre as Xbox 360 Storage (part 2)
author: error454
layout: post
permalink: /2010/09/12/using-the-palm-pre-as-xbox-360-storage-part-2/
categories:
  - PDK
  - WebOS
tags:
  - mass storage
  - pdk
  - usb
  - webos
  - xbox 360
---
<a href='http://www.usb.org/developers/devclass_docs/usb_msc_cbi_1.1.pdf'><img src='{{ site.url }}/assets/uploads/2010/09/usb.jpg' alt=''></a>
<!--more-->
<pre>Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength           39
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          4
    bmAttributes         0xc0
      Self Powered
    MaxPower              500mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           3
      bInterfaceClass         8 Mass Storage
      bInterfaceSubClass      6 SCSI
      bInterfaceProtocol      0 Control/Bulk/Interrupt
      iInterface              0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x01  EP 1 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               1
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x82  EP 2 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               8
</pre>

## Digging Deeper

I figured that snooping USB traffic might lend an insight into what's happening.  To this end I used usbmon, luckily the required module was already built-in to the stock Pre kernel.  Taking traces of both the stock driver and the modified working driver, I was very disappointed to find that the traffic for the USB detection phase was identical for both drivers?!?  How could this be?  Just a note, if you are ever looking to understand the raw text output that usbmon produces, the <a href="http://www.mjmwired.net/kernel/Documentation/usb/usbmon.txt" target="_blank">official readme</a> and the <a href="http://people.redhat.com/zaitcev/notes/usb_stern_cheat.txt" target="_blank">cheat sheet</a> are essential.

For reference, this is the repeating pattern I see in all of the usbmon traces, my decoding notes included.

<pre>//Not sure what this is but guessing that it is normal for BOT
aebf8a80 14.472869 S Ii:1:001:1 -:-606348325 2 

//Get Port 1 Status
ad8f4580 14.472930 S Ci:1:001:0 s a3 00 0000 0001 0004 4 
ad8f4580 14.472930 C Ci:1:001:0 0 4 = 07010000

//Clear Port 1 Feature 2 (over current)
ad8f4580 14.472961 S Co:1:001:0 s 23 01 0002 0001 0000 0
ad8f4580 14.472961 C Co:1:001:0 0 0

//Get Port 2 Status
ad8f4580 14.473419 S Ci:1:001:0 s a3 00 0000 0002 0004 4 
ad8f4580 14.473449 C Ci:1:001:0 0 4 = 00010000

//Get Port 3 Status
ad8f4580 14.473480 S Ci:1:001:0 s a3 00 0000 0003 0004 4 
ad8f4580 14.473480 C Ci:1:001:0 0 4 = 00010000

//Get Port 1 Status
ad8f4580 14.512908 S Ci:1:001:0 s a3 00 0000 0001 0004 4 
ad8f4580 14.512939 C Ci:1:001:0 0 4 = 03010400

//Clear Port 1 Feature 12 (ch suspend)
ad8f4580 14.512939 S Co:1:001:0 s 23 01 0012 0001 0000 0
ad8f4580 14.512939 C Co:1:001:0 0 0

//Get Device Status
ad8f4580 14.533111 S Ci:1:002:0 s 80 00 0000 0000 0002 2 
ad8f4580 14.534149 C Ci:1:002:0 0 2 = 0000

//Set Port 1 Feature 2 (over current)
ad8f4580 14.833068 S Co:1:001:0 s 23 03 0002 0001 0000 0
ad8f4580 14.833099 C Co:1:001:0 0 0

aebf8a80 14.853027 C Ii:1:001:1 -2:-606348325 0
</pre>

I feel like I'm on the wrong side of the usbmon trace!  I was hoping to find that one of the Ci requests resulted in an error code, no such luck.  Apart from errors produced from the requests bound for the Interrupt EP (Ii), things look clean.  I should note that not only is the request structure identical between the 2 drivers, but the return codes for device and port status as well.

## A Simple Working Solution

From pouring over the source for f\_mass\_storage.c, I knew that this work was based on file_storage.c by Alan Stern.  This well-documented piece of code is essentially a file-backed USB Mass Storage Device (MSD).  Meaning, when you load the module, you specify a file or device which is then presented as if it were a MSD.

I figured I may as well compile this simple module (file_storage.c) and give it a try.  Due to the way the Palm USB solution is setup, I had to do just a bit more.  It helps to understand the Palm USB ecosystem which I have painted below.

<a href='http://www.usb.org/developers/devclass_docs/usb_msc_cbi_1.1.pdf'><img src='{{ site.url }}/assets/uploads/2010/09/img_5162.jpg' alt=''></a>

The rockhopper module wraps around 5 separate USB modules, handling the loading of the desired module.  Because of this, rockhopper is always loaded in the background which means the USB ports will always be in-use.  Because rockhopper is built into the kernel by default, a recompile was necessary to compile rockhopper as a loadable module.  This allows loading my own file_storage module in place of it.

After loading the file_storage module and specifying my spare ext3fs partition (thanks to Meta-Doctor), I plugged into my Xbox 360 and was pleasantly surprised to be able to see and use my Pre as mass storage!

## file\_storage.c vs f\_mass_storage.c

Why does one module work where the other does not?  I began to look for some new tools to help me find out.  The first tool I used was Intel's USB Command Verifier, this utility lets you run a gamut of tests against your USB device, from the ones listed in Chapter 9 of the USB Spec to ones specific for MSDs.  USB CV was incredibly frustrating to get going on my 64-bit system, the process looked like this:

*   Install USB CV
*   Reboot to disable driver-signing
*   Manually install Intel EHCI debug driver due to bug in 64-bit version of USB CV
*   Go find an AT keyboard/mouse because USB device signals are being hijacked

4 reboots later and I was in business.  I ran the chapter 9 tests and the mass storage tests against both drivers.  The chapter 9 test results were identical, the mass storage results were close to identical with the following discrepencies:

**f\_mass\_storage**

*   Serial Number Test 
    *   Invalid characters in Serial Number
*   Test Case 4,8 
    *   No stall after zero-length data
*   Command Set Test 
    *   No stall after zero-length data
*   Power-Up Test 
    *   Could not find device after enumeration

I should note that I also ran the Chapter 9 and mass storage tests against every USB mass storage device I own.  2 of the devices that work with my Xbox 360 failed the Serial Number Test.  Also, I believe the Power-Up Test warning was triggered due to the amount of time it takes to plug-in the Pre and initiate mass storage mode.  All-in-all, I didn't find the smoking gun I was hoping for in these tests.

## A New Goal

I have a working solution, but I don't know why it works.  My new goal is to provide a patch for f\_mass\_storage.c that will allow it to work with the Xbox 360.  This means analyzing and understanding the changes between file\_stoage.c and f\_mass_storage.c.

Read all the answers in the <a href="http://mobilecoder.wordpress.com/2010/09/16/using-the-palm-pre-as-xbox-360-storage-the-finale/" target="_blank">finale</a>.
---
title: Using the Palm Pre as Xbox 360 storage
author: error454
layout: post
permalink: /2010/09/01/using-the-palm-pre-as-xbox-360-storage/
categories:
  - PDK
  - WebOS
tags:
  - 360
  - driver
  - linux
  - mass
  - palm
  - pre
  - storage
  - xbox
---
<img class="alignleft size-full wp-image-491" src="{{ site.url }}/assets/uploads/2010/09/usb.jpg" alt="" width="71" height="107" />I have a dream where I plug my Palm Pre into my Xbox 360 and it is recognized as a USB Mass Storage device.  I then save all my games and downloaded content to it.  Later, I fire up a PDK app that loads <a href="http://code.google.com/p/x360/" target="_blank">x360</a> (A FUSE filesystem driver for the 360) and I edit my saved games, adding more health, more gold, etc.
<!--more-->
What would it take for a USB newb to turn this into a reality?  I'm not sure, but I am willing to try.

## Spoofing Vendor/Product ID

My first thought was that the 360 was blocking Vendor/Product IDs, much like Apple's iTunes does.  So my first step was to spoof the USB Vendor/Product ID.

First, I found a USB stick that worked with the 360, I popped it into my linux box, ran lsusb -v and copied down the details.

<pre>Bus 002 Device 002: ID 1307:0163 Transcend Information, Inc. 512MB/1GB Flash Drive
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0        64
  idVendor           0x1307 Transcend Information, Inc.
  idProduct          0x0163 512MB/1GB Flash Drive
  bcdDevice            1.00
  iManufacturer           1
  iProduct                2
  iSerial                 3
  bNumConfigurations      1
</pre>

I then did the same for my Palm Pre

<pre>Bus 002 Device 003: ID 0830:0101 Palm, Inc.
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0        64
  idVendor           0x0830 Palm, Inc.
  idProduct          0x0101
  bcdDevice            2.16
  iManufacturer           1 Palm Inc.
  iProduct                2 Pre
  iSerial                 3 a5470bf024bd51193d15a9614fe7d302960c955f
  bNumConfigurations      1
</pre>

Unfortunately, the only way of changing the Vendor/Product ID is to <a href="http://www.webos-internals.org/wiki/Custom_Kernels" target="_blank">recompile the kernel</a>.  If we grep through a default config file for the Pre (look in /boot on the device), we'll find:

<pre>CONFIG_USB_ROCKHOPPER=y
CONFIG_USB_ROCKHOPPER_MANUFACTURER_STRING=Palm Inc.
CONFIG_USB_ROCKHOPPER_PRODUCT_STRING=Pre
CONFIG_USB_ROCKHOPPER_VID=0x830
CONFIG_USB_ROCKHOPPER_PID_DEV_1=0x100
CONFIG_USB_ROCKHOPPER_PID_DEV_2=0x101
CONFIG_USB_ROCKHOPPER_PID_DEBUG=0x8002
CONFIG_USB_ROCKHOPPER_PID_PASSTHRU=0x8003
CONFIG_USB_ROCKHOPPER_PID_RETAIL=0x8004
</pre>

For the curious, here is the secret decoder table:

<table border="1">
  <tr>
    <td>
      <strong>parameter</strong>
    </td>
    
    <td>
      <strong>function</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      USB_ROCKHOPPER_VID
    </td>
    
    <td>
      The USB Vendor ID
    </td>
  </tr>
  
  <tr>
    <td>
      USB_ROCKHOPPER_PID_DEV_1
    </td>
    
    <td>
      The USB Product ID for RNDIS Ethernet + Passthru
    </td>
  </tr>
  
  <tr>
    <td>
      USB_ROCKHOPPER_PID_DEV_2
    </td>
    
    <td>
      The USB Product ID for RNDIS Ethernet + Mass-Storage + Novacom.
    </td>
  </tr>
  
  <tr>
    <td>
      USB_ROCKHOPPER_PID_DEBUG
    </td>
    
    <td>
      The USB Product ID for Mass-Storage + Novacom
    </td>
  </tr>
  
  <tr>
    <td>
      USB_ROCKHOPPER_PID_PASSTHRU
    </td>
    
    <td>
      The USB Product ID for Passthru
    </td>
  </tr>
  
  <tr>
    <td>
      USB_ROCKHOPPER_PID_RETAIL
    </td>
    
    <td>
      The USB Product ID for Mass-Storage only
    </td>
  </tr>
</table>

Grep through the source tree for these config variables and you'll find all the goodies in drivers/usb/gadget/rockhopper.c.  We don't actually have to modify this source file, but long term, the elegant solution would be compiling this as a module and adding some of these config strings as arguments.

## Results of Spoofing

After compiling the kernel and loading it on the Pre, the device now looked like this:

<pre>Bus 002 Device 004: ID 1307:0163 Transcend Information, Inc. 512MB/1GB Flash Drive
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0        64
  idVendor           0x1307 Transcend Information, Inc.
  idProduct          0x0163 512MB/1GB Flash Drive
  bcdDevice            3.16
  iManufacturer           1
  iProduct                2
  iSerial                 3
</pre>

But, the 360 didn't seem to care and still couldn't see the device :( Ok, time to regroup and educate.  <a href="http://www.beyondlogic.org/usbnutshell/usb1.htm" target="_blank">USB in a Nutshell</a> became my new best friend.

## USB Configuration and Interface Descriptors

As I read about the USB spec, a couple things started nagging at me, particularly **bNumInterfaces** and Endpoint Descriptors.

USB devices can have multiple Interfaces defined.  On the Palm Pre, you can use it in Mass Storage mode, usb networking mode or Novacom mode (for debugging).  Each of these modes has an Interface Descriptor where bNumInterfaces is the total number of these Interface Descriptors.

Each Interface Descriptor contains 1 or more Endpoint Descriptors.  From an embedded background, I'm imagining these to be data pins, where each pin is configured either for input or output (source/sink).

I started taking a survey of USB devices that worked with my 360 and those that didn't work and found the following.

<table border="1">
  <tr>
    <td>
      <strong>Device</strong>
    </td>
    
    <td>
      <strong>bNumInterfaces</strong>
    </td>
    
    <td>
      <strong># of Endpoint Descriptors for Mass Storage</strong>
    </td>
    
    <td>
      <strong>Works with Xbox 360</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      Transcend flash drive
    </td>
    
    <td>
      1
    </td>
    
    <td>
      3: Bulk In, Bulk Out, Interrupt In
    </td>
    
    <td>
      yes
    </td>
  </tr>
  
  <tr>
    <td>
      Kingston flash drive
    </td>
    
    <td>
      1
    </td>
    
    <td>
      2: Bulk In, Bulk Out
    </td>
    
    <td>
      no
    </td>
  </tr>
  
  <tr>
    <td>
      Generic flash drive
    </td>
    
    <td>
      1
    </td>
    
    <td>
      3: Bulk In, Bulk Out, Interrupt In
    </td>
    
    <td>
      yes
    </td>
  </tr>
  
  <tr>
    <td>
      Palm Pre
    </td>
    
    <td>
      2 (In mass storage mode, 4 otherwise)
    </td>
    
    <td>
      2: Bulk In, Bulk Out
    </td>
    
    <td>
      no
    </td>
  </tr>
</table>

I'm not surprised that all of my flash drives have only a single interface descriptor.  The two questions I need to answer are:

1.  Does the Xbox 360 care if multiple interface descriptors exist?
2.  Does the Xbox 360 require an Interrupt Endpoint to work properly?

Well, question 1 can easily be avoided.  The reason I'm seeing 2 interfaces is because I am in developer mode (USB\_ROCKHOPPER\_PID_DEBUG).  A quick konami code later and I'm out of developer mode and down to a single interface descriptor in mass storage mode.

Question 2 will require a hack of the Pre mass storage driver.

## Hacking the Palm Pre Mass Storage Driver

Conveniently, Palm provides all of their kernel modifications as a patch.  Grep'ing through this master patch, it was quite easy to spot the interface descriptors in f\_mass\_storage.c.

When I started this little project, I knew nothing about USB other than how to plug it in.  I'm a bit further now, but to be clear, the intention of this hack is purely cosmetic.

The simple goal is:

*   Add an interrupt Endpoint for USB\_ROCKHOPPER\_PID_RETAIL

After a bit of hacking, I came up with the following patch.

<pre>--- f_mass_storage.c.orig       2010-09-01 08:27:17.223869104 -0700
+++ f_mass_storage.c    2010-09-01 20:22:28.093870603 -0700
@@ -339,6 +339,7 @@

        struct usb_ep           *bulk_in;
        struct usb_ep           *bulk_out;
+       struct usb_ep           *intr_in;

        struct fsg_buffhd       *next_buffhd_to_fill;
        struct fsg_buffhd       *next_buffhd_to_drain;
@@ -450,6 +451,8 @@
                name = bulk-in;
        else if (ep == fsg-bulk_out)
                name = bulk-out;
+       else if (ep == fsg-intr_in)
+               name = intr-in;
        else
                name = ep-name;
        FSG_DBG(fsg, %s set haltn, name);
@@ -501,7 +504,7 @@
        .bLength =              sizeof intf_desc,
        .bDescriptorType =      USB_DT_INTERFACE,

-       .bNumEndpoints =        2,              /* Adjusted during fsg_bind() */
+       .bNumEndpoints =        3,              /* Adjusted during fsg_bind() */
        .bInterfaceClass =      USB_CLASS_MASS_STORAGE,
        .bInterfaceSubClass =   US_SC_SCSI,
        .bInterfaceProtocol =   US_PR_BULK,
@@ -530,13 +533,23 @@
        /* wMaxPacketSize set by autoconfiguration */
 };

+static struct usb_endpoint_descriptor
+fs_intr_in_desc = {
+       .bLength =              USB_DT_ENDPOINT_SIZE,
+       .bDescriptorType =      USB_DT_ENDPOINT,
+
+       .bEndpointAddress =     USB_DIR_IN,
+       .bmAttributes =         USB_ENDPOINT_XFER_INT,
+};
+
 static struct usb_descriptor_header *fs_function[] = {
        (struct usb_descriptor_header *) intf_desc,
+       (struct usb_descriptor_header *) fs_intr_in_desc,
        (struct usb_descriptor_header *) fs_bulk_in_desc,
        (struct usb_descriptor_header *) fs_bulk_out_desc,
        NULL,
 };

 static struct usb_endpoint_descriptor
@@ -560,9 +573,20 @@
        .bInterval =            1,      /* NAK every 1 uframe */
 };

+static struct usb_endpoint_descriptor
+hs_intr_in_desc = {
+       .bLength =              USB_DT_ENDPOINT_SIZE,
+       .bDescriptorType =      USB_DT_ENDPOINT,
+
+       /* bEndpointAddress copied from fs_bulk_out_desc during fsg_bind() */
+       .bmAttributes =         USB_ENDPOINT_XFER_INT,
+       .wMaxPacketSize =        __constant_cpu_to_le16(64),
+       .bInterval =            8,
+};

 static struct usb_descriptor_header *hs_function[] = {
        (struct usb_descriptor_header *) intf_desc,
+       (struct usb_descriptor_header *) hs_intr_in_desc,
        (struct usb_descriptor_header *) hs_bulk_in_desc,
        (struct usb_descriptor_header *) hs_bulk_out_desc,
        NULL,
</pre>

Every time you hack a USB endpoint, a USB programmer dies :/  Sorry!

## Mass Storage Driver Hack Results

After recompiling and loading the mass storage driver, the endpoints look like this:

<pre>Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           3
      bInterfaceClass         8 Mass Storage
      bInterfaceSubClass      6 SCSI
      bInterfaceProtocol     80 Bulk (Zip)
      iInterface              0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x00  EP 0 OUT
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               8
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
</pre>

The interrupt endpoint is there but the fact that it is enumerated as EP0 is a problem, EP0 is reserved and not supposed to have a descriptor!   Clearly I am missing something here!  I took several passes at this, trying to get the interrupt endpoint recognized as 082 / EP2, which is used in other device configurations as an interrupt endpoint.  So far none have sticked.

As I build up my understanding of the composite and gadget usb frameworks, I found a good article on the <a href="http://lwn.net/Articles/395712/" target="_blank">composite usb framework</a> which unfortunately shows that most folks writing USB mass storage drivers in linux leave the endpoint descriptor definitions alone.  The endpoint details happen behind the scenes and get implemented as part of the framework rather than being defined by your driver.  This project is getting put on the back burner for now until I have some quality time to devote to understanding the frameworks involved.

Continued in <a href="http://error454.com/2010/09/12/using-the-palm-pre-as-xbox-360-storage-part-2/" target="_blank">Part 2.</a>
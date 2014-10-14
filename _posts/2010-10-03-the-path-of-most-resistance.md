---
title: The Path of Most Resistance
author: error454
layout: post
permalink: /2010/10/03/the-path-of-most-resistance/
categories:
  - Ares
  - PDK
  - WebOS
tags:
  - palm pre
  - proc
  - ps3
  - sysfs
  - usb
  - wii
  - xbox
---
<table>
  <tr>
    <td>
      <a href=''><img src='{{ site.url }}/assets/uploads/2010/10/519x361.jpg?w=300' alt=''></a>
      
      <p>
        I am building a track record of choosing projects that somehow flex the limits of what is easily doable on the Palm Pre.
      </p>
      
      <p>
        First I wanted to use the Palm Pre as a mass storage device on my Xbox 360.  The unraveling of that rabbit-hole took a full month.  Now I want to be able to create and mount regular files as USB devices on the Palm Pre (an app I'm calling USB Mass Storage Tool).  This time I am hitting a technical limitation imposed by Palm.
      </p>
      
      <p>
        The current limitation is that sysfs is not available for applications to read/write from within the jailed environment in which apps run.</td> </tr> </tbody> </table> <h2>
          The Need
        </h2>
        
        <p>
          <strong>Q.</strong> Why would anyone need USB Mass Storage Tool?  I mean, the Pre now works with the Xbox 360, isn't that enough?
        </p>
        
        <p>
          <strong>A.</strong> It is not enough.  When you plug the Pre (or any storage device) into the Xbox 360 and attempt to use the device for data storage (storing saved games/profile information), the Xbox 360 will format the entire device.  For the Palm Pre owner this means you lose /media/internal.
        </p>
        
        <p>
          If you want to cry, try this.  Hook your Pre into the Xbox, format it to be used as Xbox storage and then unplug your device.  You will notice that all of your apps, photos, videos, etc are forever gone.
        </p>
        
        <h2>
          The Dream
        </h2>
        
        <p>
          My dream is to be able to store a USB image for my Wii, Xbox 360 and PS3 as an actual file on the Pre (I don't own a PS3, but PS3 owners should be interested in this).  So when you browse the files on my device you will see:
        </p>
        
        <p>
          xbox.img<br /></br> wii.img<br /></br> ps3.img
        </p>
        
        <p>
          When I want to save games on my Xbox, I choose xbox.img and present this file as my USB storage device.  The Xbox formats the entire device (which is just a file sitting safely on /media/internal) and everyone walks away with no data loss.
        </p>
        
        <table border="0">
          <tr>
            <td>
              <img class="size-full wp-image-605 alignnone" title="create image" src="{{ site.url }}/assets/uploads/2010/10/massstoragetool_2010-03-10_201126.png" alt="" width="320" height="480" />
            </td>
            
            <td>
              <img class="alignnone size-full wp-image-611" title="mounting an image" src="{{ site.url }}/assets/uploads/2010/10/massstoragetool_2010-03-10_204331.png" alt="" width="320" height="480" />
            </td>
          </tr>
        </table>
        
        <h2>
          Realizing the Dream
        </h2>
        
        <p>
          Not only is this possible, it is incredibly easy to do.  You can mount any file or device by echoing said file/device into an entry in the sysfs filesystem:
        </p>
        
        <pre>
echo /media/internal/xbox.img  /sys/devices/platform/musb_hdrc.0/gadget/gadget-lun0/file
</pre>
        
        <p>
          This sysfs entry was created by f_mass_storage.c for this very purpose.  However, since Palm does not allow apps to access sysfs, what would have been a simple app has once again turned into usb module hacking.  The sysfs filesystem contains non-process related information about drivers and hardware.  Preventing user-level access to sysfs while allowing access to procfs doesn't make a lot of sense from a security perspective.  I have asked Palm for details on whether they plan on changing this policy.
        </p>
        
        <p>
          In the meantime, because Palm allows access to /proc, I am going to modify the USB Mass storage driver so that it creates a procfs entry on module startup.  This procfs entry will mimic the behavior of the sysfs entry.
        </p>
---
title: Using the Homebrew Javascript Service Framework
author: error454
layout: post
permalink: /2011/03/18/using-the-homebrew-javascript-service-framework/
categories:
  - Homebrew
  - WebOS
tags:
  - framework
  - homebrew
  - js
  - node
  - node.js
  - root
  - service
  - webos
---
<a href=''><img src='{{ site.url }}/assets/uploads/2011/03/out-of-jail.jpg' alt=''></a>

WebOS 2.x allows us to create services using node.js.  Services are great, but by default they run in a jail that prevents them from accessing the entire device.  For many services this is fine because there is no need to do anything outside of this jail.  On the other hand, there are some services that are only useful if they have root access.  To obtain root access, these services can use Jason Robitaille's <a id="project_summary_link" href="http://code.google.com/p/homebrew-js-service-framework/">Homebrew Javascript Service Framework for webOS</a>, hereto referred to as HJSF.
<!--more-->
This article is going to explore the essential configuration requirements and validation steps for using HJSF in a node.js service.



## Requirements

To correctly utilize HJSF, 3 requirements must be met.

### Requirement 1  HJSF Must Be Installed

Get it from Preware or download the ipk from the google code project page.

### Requirement 2  Custom dbus File

A file named **dbus **should exist in the root of your services source tree, at the same level as your service assistant.  The contents of the dbus file are as follows:

<pre>[D-BUS Service]
Name=APPID
Exec=/var/usr/bin/run-homebrew-js-service /media/cryptofs/apps/usr/palm/services/APPID</pre>

Where APPID is the id of your application, as an example, the dbus entry for my mass storage service (com.wordpress.mobilecoder.umst.service) would be:

<pre>[D-BUS Service]
Name=com.wordpress.mobilecoder.umst.service
Exec=/var/usr/bin/run-homebrew-js-service /media/cryptofs/apps/usr/palm/services/com.wordpress.mobilecoder.umst.service</pre>

### Requirement 3  Install Scripts

The primary purpose of the install scripts is to copy the dbus file to the proper location with the proper name.  The scripts handle copying the dbus file to /var/palm/ls2/services/pub and /var/palm/ls2/services/prv.  Be careful, your dbus file must be named **APPID.service** for this to work.  This means that if your APPID is **com.joeblow.service** the resulting dbus file must be named **com.joeblow.service.service**.  You can see how this might be confusing if your ending prefix is .service.

<table border="1">
  <tr>
    <td>
      <strong>APPID</strong>
    </td>
    
    <td>
      <strong>dbus filename</strong>
    </td>
    
    <td>
      <strong>Correct?</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      com.joeblow.service
    </td>
    
    <td>
      com.joeblow.service
    </td>
    
    <td>
      WRONG
    </td>
  </tr>
  
  <tr>
    <td>
      com.joeblow.service
    </td>
    
    <td>
      com.joeblow.service.service
    </td>
    
    <td>
      CORRECT!
    </td>
  </tr>
</table>



Below, I have posted the 4 install scripts for my <a href="http://gitorious.org/usb-mass-storage-tools-service" target="_blank">Mass Storage Tools Service</a>.  You can use these as examples, I believe you'd be hard pressed to find a more barebones service to copy from.  I used Jason's <a id="project_summary_link" href="http://code.google.com/p/systoolsmgr-service/">SysToolsMgr Service</a> as a reference to base my scripts on.  To be clear, if you replace the ID below with your app ID, the scripts will handle naming your dbus file correctly.

prerm / pmPreRemove

<pre>#!/bin/sh

ID=com.wordpress.mobilecoder.umst.service

#remount root using technique that won't cause the random remounting error
if [ -z $IPKG_OFFLINE_ROOT ]; then
	/usr/sbin/rootfs_open -w
fi

#remove dbus service file
/bin/rm -f /var/palm/ls2/services/prv/$ID.service
/bin/rm -f /var/palm/ls2/services/pub/$ID.service

exit 0
</pre>

postinst / pmPostInstall

<pre>#!/bin/sh

ID=com.wordpress.mobilecoder.umst.service
SERVICES_PATH=/media/cryptofs/apps/usr/palm/services/$ID

if [ -z $IPKG_OFFLINE_ROOT ]; then
	/usr/sbin/rootfs_open -w
fi

#make directories in the rare event they don't exist
/bin/mkdir -p /var/palm/ls2/services/prv
/bin/mkdir -p /var/palm/ls2/services/pub

#copy dbus service file
/bin/cp -f $SERVICES_PATH/dbus /var/palm/ls2/services/prv/$ID.service
/bin/cp -f $SERVICES_PATH/dbus /var/palm/ls2/services/pub/$ID.service

exit 0
</pre>

## Validation

Once you pull a basic service together, you will want to validate that you have root access.  One simple way to do so, recommended by Jason is to run the **id** system command.  To run this command, you will need to grab Jason's CommandLine.js file from the SystoolsMgrService and add it to your sources.json. A fictitious service assistant that runs the id command might look like this:

<pre>var MyAssistant = function(){
}

MyAssistant.prototype.run = function(future) {
	//Using the commandline we can determine what ID we are running as.
	//You will need to print this value from your companion application
	this.cmd = new CommandLine(id, this.future);
	this.cmd.run();
}
</pre>

The desired output would show that you are running with a UID and GID of root.

## Stuck?

There are plenty of places along the way to get stuck, if you find yourself stuck, just leave a comment with your issue and we can enrich the community with an answer.
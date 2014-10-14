---
title: Introducing ShiVa Android All In One (AAIO)
author: error454
layout: post
permalink: /2012/10/22/introducing-shiva-android-all-in-one-aaio/
publicize_twitter_user:
  - error454
categories:
  - Android
  - Shiva 3D
tags:
  - all in one
  - android
  - open source
  - sdk
  - shiva
---
## What is this?

This is an open source project that I've started to help ShiVa developers get the latest/greatest Android SDK implementations.  The goal for this project is to be a single Android project that contains the implementations for any Android SDK that you could possibly care about.

There are 2 pieces to this project.  The Android code is where the SDK implementations live as combinations of Java and JNI (for interacting with ShiVa).  The ShiVa project is where SDK specific AI implementations go.  So for every SDK implemented, there is an associated ShiVa AI model that exposes the functionality needed to use the SDK from inside ShiVa.

## Why did I make it?

I often see folks posting large tutorials in the forums detailing how to implement various SDKs in Android.  These tutorials are awesome and the community members involved in writing them deserve tons of credit.  What happens when you start a new project though?  Do you comb back over the forums, find the 3 SDK tutorials of interest and start copying/pasting all over again?  What happens when the SDK is updated with breaking changes and the tutorial author is busy?

Wouldn't it be nice if you didn't have to copy/paste any of this code?  Wouldn't it be wonderful if all you had to do was select the SDKs that you wanted to use?  I answer yes to these questions and this is why I have started this project.



## Why should you use it?

As a ShiVa user, what would compel you to use this?

*   Easier integration  Instead of copying/pasting code, you only have to configure the SDKs you want to use.
*   Oversight  More than a single person is looking at the implementation for any given SDK to spot potential issues, this leads to better code.
*   Bug filing and enhancement requests  Besides integrating SDKs, this is a project where we can patch up the UAT generated code if needed (it already has a couple fixes).  Anything from device specific workarounds to fixing outright bugs.

## Prerequisites

Before getting started, I am assuming that you've done the following:

1.  Installed <a href="http://www.oracle.com/technetwork/java/javase/downloads/index.html" target="_blank">JDK 6</a>
2.  Installed Eclipse 3.6.2 or greater
3.  Installed the <a href="http://developer.android.com/sdk/index.html" target="_blank">Android SDK</a>
4.  Downloaded the latest Android 4.1 dev tools from Android SDK Manager
5.  Installed the <a href="http://developer.android.com/tools/sdk/ndk/index.html" target="_blank">Android NDK</a>
6.  Installed the <a href="http://developer.android.com/tools/sdk/eclipse-adt.html" target="_blank">ADT plugin</a>
7.  Installed <a href="http://ant.apache.org/" target="_blank">Apache Ant > 1.8</a>
8.  Installed Cygwin
9.  Configured your system paths to be able to build in Eclipse

Unfortunately there's not an easy way to improve on the Developer eXperience for these essential requirements.  If your existing exported ShiVa projects aren't compiling in Eclipse, don't expect AAIO to compile either.

Please do not comment here asking for build help, post on the ShiVa Android forum instead, the build process has not changed in AAIO.

## How to Download, Configure & Build

Here is a full run-thru of how to get started using AAIO.

### 1. Download the source

The source is on the <a title="aaio" href="https://github.com/error454/ShiVa-Android-All-In-One" target="_blank">github page</a>.  You can download it by clicking the zip button.

<a href=''><img src='http://mobilecoder.files.wordpress.com/2012/10/download.jpg' alt=''></a>

Once you unzip the project, you will have 2 folders:

1.  **android**  The eclipse project.
2.  **shiva**  The ShiVa project that contains all of the necessary AI Models along with a simple UI that allows testing the SDK integration.  This is also compiled as S3DMain.smf in the eclipse/assets folder.

### 2. Rename the Source

This step renames the namespace of the source files and updates the build scripts with the correct NDK location on your system.  Obviously you don't want your project namespace to be com.wordpress.mobilecoder.aaio and Android All In One right?  To begin, open up the android/configure.sh file, I recommend using <a href="http://notepad-plus-plus.org/" target="_blank">Notepad++</a> to edit this file, especially make sure that under **Edit->EOL Conversion** it is set to **Unix**.  If you try to edit this in windows notepad and save it out, cygwin is going to have issues when you try to execute the script.

Here is the portion of the file we're concerned with.

<pre>#####REQUIRED VALUES#####
#The name of your package.
#Example: com.atari.frogger
newPackage="default"
#The name of your project class, this cannot contain spaces.
#Example: froggerClass
newName="default"
#The title of your project, this can contain spaces and is what is displayed on the Android home screen.
#Example: "Frogger Extreme"
newFriendlyName="default"
#The absolute path that contains the ndk-build command from the Android NDK
#Example: "/cygdrive/C/sdks/android-ndk-r7/"
newNDKPath="default"
#####END REQUIRED VALUES#####</pre>

We need to edit all of the entries that don't start with a hash sign.  After finding the location of the NDK on my system, I change the values to look like this (don't forget the trailing slash on the ndk path!)

<pre>#####REQUIRED VALUES#####
#The name of your package.
#Example: com.atari.frogger
newPackage="com.example.aaio"
#The name of your project class, this cannot contain spaces.
#Example: froggerClass
newName="example"
#The title of your project, this can contain spaces and is what is displayed on the Android home screen.
#Example: Frogger Extreme"
newFriendlyName="An awesome AAIO Example"
#The absolute path that contains the ndk-build command from the Android NDK
#Example: "/cygdrive/C/sdks/android-ndk-r7/"
newNDKPath="/cygdrive/C/sdk/android-ndk-r8b/"
#####END REQUIRED VALUES#####</pre>

Finally, open up cygwin and browse to the path where the file lives and run the script

<a href=''><img src='{{ site.url }}/assets/uploads/2012/10/cygwin.jpg' alt=''></a>

*Note for the advanced: If you ever need to rename your project again, you can edit the lower portion of the script with the new values you just entered.*

### Import into Eclipse

Fire up eclipse and select **File->Import**

Select Existing Android Code Into Workspace:

<a href=''><img src='http://mobilecoder.files.wordpress.com/2012/10/existing-code.jpg' alt=''></a>

Browse to the same folder where the configure.sh script was and hit ok, you should see this (although obviously with whatever values you chose) at which point you can click finish:

<a href=''><img src='http://mobilecoder.files.wordpress.com/2012/10/import.jpg' alt=''></a>

### Copy the ShiVa Libs

The ShiVa libs are not included in this project, to get them you must:

1.  Export an *android project* from the Authoring Tool
2.  In the exported project, copy the **obj **folder to your new project and overwrite the existing **obj** folder

*If Stonetrip permits it, I will include this in the future (hint hint)*

### Building

Building works like it always has.

1.  Drag build.xml into the ant view
2.  Double click a build

<a href=''><img src='http://mobilecoder.files.wordpress.com/2012/10/build.jpg' alt=''></a>

### Enabling SDKs etc

Now that you can build, you are ready to start enabling SDKs etc.  Please see the <a href="https://github.com/error454/ShiVa-Android-All-In-One#how-to-enableconfigure-sdks-in-eclipse" target="_blank">following section of the README</a>.

## Final Hint

If you've updated your ShiVa project and need to bring it into eclipse, here is all you need to do:

1.  Export as an STK file from ShiVa editor
2.  Rename the stk file to S3DMain.smf
3.  Copy **S3DMain.smf** into your **assets** folder, overwriting the previous file
4.  Build!
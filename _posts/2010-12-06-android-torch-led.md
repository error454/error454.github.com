---
title: Android Torch LED
author: error454
layout: post
permalink: /2010/12/06/android-torch-led/
categories:
  - Android
tags:
  - android
  - camera
  - flash
  - led
  - light sensor
  - torch
---

<img class="size-full wp-image-689 alignnone" src="{{ site.url }}/assets/uploads/2010/12/human-torch.jpg" alt="" width="265" height="210" />

    
Many Android devices have an LED next to the camera lens that can be used as a flash.  I've been doing some research on the so-called Torch mode of this LED.  Torch mode simply turns the LED on and leaves it on as opposed to temporarily flashing the LED during a photo.I'm sure everyone has seen the flashlight apps that do just this. I am looking at possible scenarios where the torch could help with image analysis.  A secondary goal is to have the LED light enabled automatically based on measurable conditions.



# The Torch API

The API calls for using the torch are incredibly simple.  The Android <a href="http://developer.android.com/reference/android/hardware/Camera.html" target="_blank">Camera API</a> contains a <a href="http://developer.android.com/reference/android/hardware/Camera.Parameters.html" target="_blank">Camera.Parameters</a> class that is used to change the parameters for the camera.   A parameter called <a href="http://developer.android.com/reference/android/hardware/Camera.Parameters.html#FLASH_MODE_TORCH" target="_blank">FLASH_MODE_TORCH</a> was implemented in API level 5 (Android 2.0).

To turn the torch on, you simply set the camera parameter Camera.Parameters.FLASH\_MODE\_TORCH: (side-note, these code snippets are not production ready by a long shot)

<pre>Camera mCamera;
Camera.Parameters mParameters;

//Get a reference to the camera/parameters
mCamera = Camera.open();
mParameters = mCamera.getParameters();

//Set the torch parameter
mParameters.setFlashMode(Camera.Parameters.FLASH_MODE_TORCH);

//Comit camera parameters
mCamera.setParameters(mParameters);
</pre>

To turn the torch off, set Camera.Parameters.FLASH\_MODE\_OFF

# Device Support

Before using the torch, be sure that the phone supports it.   There are many reports of devices whose camera driver does not return FLASH\_MODE\_TORCH as a supported option.   Many of these devices can still turn the torch on but cannot turn it off without releasing the camera object.

\*EDIT\*

In my recent testing, I have come across at least one phone (Droid X) that has very poor torch implementation.  This is no doubt a bug somewhere in the camera driver which makes the flash cycle on/off when autofocus is engaged.  I have [reported this to Motorola][1] and hope to see it fixed in the future.

Another caveat is that you cannot change camera parameters while autofocus is running.  In my mind, this exposes a huge deficiency in the Android API.  You see, there is no mechanism that tracks autofocus state in the API.  You have to set a flag when you call autofocus and clear it in the autofocus callback.  Oh, and don't try canceling autofocus as a shortcut  the autofocus callback will never get called and you will have an unknown delay before the asynchronous cancelAutofocus actually finishes (naturally there isn't a callback for this either).

A sample that checks for torch support might look something like this:

<pre>...inside some class
//Create camera and parameter objects
private Camera mCamera;
private Camera.Parameters mParameters;
private boolean mbTorchEnabled = false;

//... later in a click handler or other location, assuming that the mCamera object has already been instantiated with Camera.open()
mParameters = mCamera.getParameters();

//Get supported flash modes
List flashModes = mParameters.getSupportedFlashModes ();

//Make sure that torch mode is supported
//EDIT - wrong and dangerous to check for torch support this way
//if(flashModes != null  flashModes.contains(torch)){
if(flashModes != null  flashModes.contains(Camera.Parameters.FLASH_MODE_TORCH)){
	if(mbTorchEnabled){
		//Set the flash parameter to off
		mParameters.setFlashMode(Camera.Parameters.FLASH_MODE_OFF);
	}
	else{
		//Set the flash parameter to use the torch
		mParameters.setFlashMode(Camera.Parameters.FLASH_MODE_TORCH);
	}

	//Commit the camera parameters
	mCamera.setParameters(mParameters);

	mbTorchEnabled = !mbTorchEnabled;
}
</pre>

# Controlling Torch Brightness

The Camera API does not allow you to control the brightness of the torch mode.  I did some prototyping to test the possibility of controlling torch brightness by using pulse width modulation (PWM). Unfortunately, I was unable to toggle the duty cycle fast enough to maintain persistance of vision.  Even at 120 Hz and full duty cycle, the flickering was noticeable to the naked eye which means it was likely clocking in at under 60 Hz.

I have to believe that this delay is either due to the overhead of the function calls or a hardware induced delay.  Needless to say, unless you have direct control to drive the LED, PWM is not a viable option for controlling the brightness level of the Torch.

# Torch Activation Criteria

A secondary goal was the possibility of automatically enabling the torch based on certain criteria.

## Ambient Light

It made sense that the light-sensor on the device would be a good fit to accomplish automated torch activation.

Android devices have a built-in ambient light sensor, the current generation of devices have front-facing light sensors only.  An application can register to be notified when there is a change in light sensor data.  I wrote a prototype to display the current ambient light-level and then tested the lux accuracy in known lighting conditions.

The light sensors in the devices I tested (Nexus One & a few Samsung devices) were not very accurate, the biggest issue is the level of quantization from the sensor.  The lowest discrete level below 160 lux is 10 lux.

Because the API only notifies you when a change in light-level has occurred based on discrete intervals, this means that a reading of 10 lux followed by a reading of 159 lux wouldn't produce a notification.  The indoor lighting conditions I am targeting are around 10-100 lux, clearly the sensor data is not sufficient in this scenario.

Oh well, the sensor was on the wrong side of the device to begin with.

## Image Analysis

I briefly considered using image data to control torch activation.   This would be prone to the same pitfalls that the camera world has been solving for the last 25+ years  handling complicated scenes with odd mixes of dark or white colored objects (snow and dark-skinned people to name a few).

## User Activation

Definitely the easiest option, just give the user the option to toggle the torch.   The concern here is whether there are a set of scenes where the torch would degrade the  reading experience.

 [1]: http://bit.ly/fOMCfU
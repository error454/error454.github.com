---
title: "Stormtrooper's Guide to Droid JTAG'ing"
author: error454
layout: post
permalink: /2013/12/19/stormtroopers-guide-to-droid-jtaging/
categories:
  - Android
  - Hacking
  - Interesting Problems At Work
tags:
  - android
  - droid
  - flux
  - gauge
  - gsm
  - jtag
  - jtaging
  - lift
  - pad
  - pins
  - solder
  - stormtrooper
  - torx
  - tutorial
  - wire
---
<h2 style="text-align: center;">
  TOP SECRET  EMPIRE PERSONNEL ONLY
</h2>

You've done it!  You've finally found the droids you were looking for.  But hey, wait a minute, they don't even power on?!? How are you supposed to re-purpose them for your evil schemes and recover all those juicy princess holocrons?

Buckle-up soldier! It's time to JTAG!
<!--more-->
### What's JTAG?

JTAG is a debugging interface that pretty much every CPU has on it, even droids!  Once you hook up, you can read/write to flash memory and change out those bad motivators!

### Step 1. Gather Tools

<a href='{{ site.url }}/assets/uploads/2013/12/tools1.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/tools1-1024x768.jpg' alt='TK-454 with some required tools.'></a>

There are some necessary tools to get this job done right.

*   Torx screwdriver (usually t5 or t6)
*   Solder (no larger than 0.022)
*   Flux
*   Soldering station
*   Wet sponge

It's important to use a temperature controlled soldering station.  We recommend a temperature of ~ 700° F.

<a href='{{ site.url }}/assets/uploads/2013/12/iron-heat.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/iron-heat-1024x768.jpg' alt='You thought you could use a $15 non-temperature controlled soldering iron? I bet you thought you&apos;d just run down to Tosche Station and pick up some power converters instead of doing your chores too huh?'></a>

### Step 2. Remove the Case

Don't be fooled by this step.  The Rebel Scums have put plenty of traps in your path to make you fail.  Follow these guidelines and keep your blaster ready!

1.  Keep screws organized
2.  If the plastic isn't budging, look for a missed screw
3.  Every time a plastic piece is removed, look for more screws
4.  Use a small flat-head screwdriver to help slide the case open

<a href='{{ site.url }}/assets/uploads/2013/12/unscrew1.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/unscrew1-1024x768.jpg' alt='TK-454 carefully unscrewing the Torx screws'></a>

Be warned!  When the case comes off, 2 plastic pieces will fall out as if something has broken.  Tell your armed escort about this ahead of time to avoid any potential confusion.  Then put the pieces aside until needed for reassembly.  If by accident you do break a piece of the case, you may want to google: <a href="https://www.google.com/search?q=stormtrooper+helmet+with+anti-force+choke" target="_blank">Stormtrooper helmet with anti force choke</a>.

<a href='{{ site.url }}/assets/uploads/2013/12/pieces.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/pieces-683x1024.jpg' alt='Those pieces are supposed to be there, don&apos;t tell Lord Vader!'></a>

### Step 3  Disconnect Motherboard Cables

With the case off, you can now disconnect the motherboard cables.  There are usually 2 or 3 cables that need to be removed.  Just flip that little white lever and the cable slides right out.  Keep limbs and weapons free and clear of the USB port.

<a href='{{ site.url }}/assets/uploads/2013/12/ribbon-cables1.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/ribbon-cables1-1024x768.jpg' alt='Detach all of the ribbon cables from the motherboard.'></a>

Once all ribbon cables are removed, you can detach the motherboard from the screen. Use proper lifting techniques as illustrated below.  Optionally, you may equip utility belt UP-37 for more lifting support.

<a href='{{ site.url }}/assets/uploads/2013/12/screen1.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/screen1-1024x683.jpg' alt='Lift the motherboard off the screen to complete dis-assembly.'></a>

### Step 4. Reveal JTAG Pins

Once you flip the motherboard over, don't be fooled by the deception of the rebellion!  Fire a few blaster shots into any mysterious adhesives you find.  If you are unable to find the JTAG pins, load training module <a href="https://www.google.com/search?site=&tbm=isch&source=hp&biw=1858&bih=1071&q=jtag+pinout+%5Byour+phone+here%5D&oq=jtag+pinout+%5Byour+phone+here%5D&gs_l=img.3...1625.11891.0.12969.35.16.3.16.0.1.110.813.15j1.16.0....0...1ac.1.32.img..22.13.547.X_xfShDHp1s" target="_blank">JTAG Holosearch</a>.

<a href='{{ site.url }}/assets/uploads/2013/12/black-tape1.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/black-tape1-1024x768.jpg' alt='Those Rebel Scum thought they could hide the JTAG pins!'></a>

### Step 5. Connect to JTAG Pins

OK trooper, there are a lot of options here.  Depending on how much training you have, you could go one of several routes.  Be sure to check with our<a href="http://gsmserver.com/" target="_blank"> Intergalactic Provider</a> regarding some of the pre-made JTAG units available.

Maybe you're using a raspberry PI or maybe you have a custom JTAG box like Medusa or RIFF.  Worst case, you'll have to solder all the JTAG pins yourself.  If you're soldering yourself, be sure to always flux the pads and then tin them.  The power of the flux rivals the powers of the light and dark-side combined!

<a href='{{ site.url }}/assets/uploads/2013/12/flux.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/flux-1024x768.jpg' alt='Use the flux TK!'></a>

If you haven't yet completed module: *The Zen of Force Based Soldering *then here are some helpful tips:

1.  Use flux!
2.  Use a small amount of solder on the tip of the iron to conduct heat to the pin pad
3.  Tin the pad before trying to solder a wire to it
4.  Heat a tinned pad with a small amount of solder on the tip and then push the wire into the solder

You should never use large gauge wire to solder to the phone PCB, doing so will put too much strain on the joint and will result in a lifted pad.  You will eventually lift a pad, it happens, better to practice your skills on some old R1 units before doing the real deal.

<a href='{{ site.url }}/assets/uploads/2013/12/soldering.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/soldering-1024x683.jpg' alt='Use small wires and a clean tip when soldering to the pin pads.'></a>

If you only ever have to JTAG a single unit, then soldering wires will get the job done.  But, most of us vets use JTAG jigs.  They're a specially made PCB that is pressed onto the JTAG contacts using a clip.  These units are safer (no lifted pads), involve little or no soldering and they make JTAG'ing multiple units a snap!

You could make your own if you have time or you can buy them pre-made especially for your device and JTAG hardware combination.  They cost slightly less than a stein of grog at your local cantina.

<a href='{{ site.url }}/assets/uploads/2013/12/jtag-clip.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/jtag-clip-1024x768.jpg' alt='JTAG clips reduce chances of failure.'></a>

When JTAG units are connected and working, do not bump, breath or look at them wrong!

<a href='{{ site.url }}/assets/uploads/2013/12/horseplay.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/horseplay-1024x768.jpg' alt='Horseplay is prohibited around the JTAG adapter.'></a>

### Step 6  Cleanup

Clean up the extra flux with some rubbing alcohol before you power up.

<a href='{{ site.url }}/assets/uploads/2013/12/cleanup.jpg'><img src='{{ site.url }}/assets/uploads/2013/12/cleanup-768x1024.jpg' alt='Clean up that excess flux with some rubbing alcohol.'></a>

### Step 7  Commence with Evil Plans

Looks like you're all set to go.  Good luck with that R2 unit :)
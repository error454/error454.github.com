---
title: The 5 Golden Rules of Input
author: error454
layout: post
permalink: /2015/10/18/5/golden/rules/of/input
categories:
  - game-theory
  - gamedev
tags:
  - gamepad
  - input
  - controller
  - mapping
  - geometry wars
  - pacman
  - child of light
  - assault android cactus
  - tomb raider
  - lara croft
  - joystick
---

Imagine for a moment that you are out car shopping. As you reach out to open a car door you realize there's no handle?!? The salesman says "oh yeah, you have to crawl through the window in this one"...

Doors that open, this is a car feature that everyone has come to expect in consumer vehicles. Similarly, there is a base feature set that players are going to expect from your game. If you don't hit this base feature set, players could exit your game early out of frustration before they even see the cool stuff.

This article will give you easy to follow rules on how to support gamepad and mouse/keyboard like a world boss. To do this we're gonna do 3 things, first we're going to talk about the high-level logic of detecting which input device the player is using. Next we're going to cover the 5 rules that will provide a great player input experience. Finally, we're going to see how a few AAA games hold up against these rules.

This article is primarily aimed at single player PC games that support gamepads, so if you're doing couch co-op or splitscreen you will need to take this with a grain of common sense.

<!--more-->

# The Input Selection Engine #
Let's state the problem we're trying to solve. The goal is to support both gamepads and mouse/keyboard control in our game. The player should be able to control the game using any input device by simply using that input device. This means no getting off the couch or hunting through menus to toggle a gamepad option.

To achieve this goal, we need an engine (a piece of code that runs in a loop) that will tell us which input device the player wants to use. The engine logic can be broken down by answering two simple questions:

1. Does the player want to use the gamepad?
2. Does the player want to use the mouse and keyboard?

## Does the Player want to use the Gamepad? ##
Consider this chart, where the green check mark is a 100% indicator, the red X is a 0% indicator and the yellow exclamation is somewhere in-between.

<img src='{{ site.url }}/assets/uploads/2015/10/gamepadevents.png'>

Remember that many people have hard-wired gamepads that are always connected to the PC. So seeing a detected gamepad isn't a conclusive indicator of the player's desire to use a gamepad.

The reason that the analog stick is not a definite yes is that it's possible that the player has a floaty analog stick. So while you might completely ignore this and just treat it as an absolute, it's a good idea to define an analog noise floor and ignore values below it. Your engine might be doing this already.

## Does the Player want to use the Keyboard/Mouse? ##
Consider this chart.
<img src='{{ site.url }}/assets/uploads/2015/10/keyboardevents.png'>

The mouse movement falls into the same cautionary category as analog joystick movement. It's a good indicator that the player wants to use the mouse but there's also a small chance of jitter (like when a cat hair gets stuck under the laser). The easy solution is setting a minimum distance threshold for mouse movement.

## Engine Logic ##
Based on the charts above, we can write an engine that toggles between gamepad and keyboard/mouse control on the fly. This will always be running in the background and will automatically switch to the preferred input device without missing a beat.

I call this the Timestamp Input Engine. The main idea is that you store 2 timestamps that indicate the time when the last input event was received for both gamepad and keyboard/mouse. Every time you detect a valid input event, you update the appropriate timestamp variable with the current game time. Make sure to use elapsed game time, not delta-time or a time that might get paused or dilated.

<img src='{{ site.url }}/assets/uploads/2015/10/timestamps.png'>

Now that you have timestamps indicating the last gamepad and keyboard events, the actual input detection is as easy as seeing which timestamp is greater.

<img src='{{ site.url }}/assets/uploads/2015/10/tie.png'>
 

# The 5 Rules #
Now that our engine is defined, let's focus on satisfying what I'm calling The 5 Golden Rules of Input. I feel that these rules are key to allowing the player to have a non-frustrating single player experience.

## Rule 1 - Icons Match the Input Device   
Whenever the input device is changed, all on-screen button icons should change to match the corresponding device. You can see an example of this in geometry wars by popping the battery out of your gamepad.

<img src='{{ site.url }}/assets/uploads/2015/10/geo1.jpg'>

<img src='{{ site.url }}/assets/uploads/2015/10/geo2.jpg'>

## Rule 2 - Mouse Cursor Matches the Input Device ##
This should go without saying that if the player is using a gamepad, they don't want a mouse cursor in the middle of the screen, you know, just sitting there ruining their life. If they *are* using a mouse then they probably want to see the mouse cursor.

With Steam machines and Steam big picture mode, many players with gamepads are on a couch. Don't make them stand up and walk over to the PC to drag the mouse cursor to the bottom of the screen. Also, when you hide the mouse, make sure that you aren't just changing the visibility of the cursor but are also disabling the ability to generate selection/hover events in menus.

## Rule 3 - All Devices Work 100% of the Time ##
At all times, the player must be able to navigate all menus and control the game with any connected input device. And without explicitly selecting the device in a menu.

## Rule 4 - DPAD, Analog Stick & Mouse can Navigate Menus##
The player must be able to navigate menus using the mouse and both the DPAD and analog stick. Not only is this a good player experience but it will also allow you to market your game as having Full Controller Support on Steam.

## Rule 5 - A Disconnected Gamepad Pauses the Game ##
When the player is using a gamepad and the gamepad gets disconnected, the game should pause. Probably the only caveat to this is if the player is in a menu, in which case the input device should simply toggle. Be sure that the pause screen follows Rules 1-4.

Child of Light is a good example of this in action, you can see them following Rules 1 & 5 here.

<img src='{{ site.url }}/assets/uploads/2015/10/col1.jpg'>

# How Games Measure Up #
Now it's time to check out a few of the gamepad enabled games in my steam account and see how they measure up to these 5 rules.

<img src='{{ site.url }}/assets/uploads/2015/10/gamechart.png'>

## Geometry Wars 3 ##
The main issue with GW3 is a design issue that many games suffer from. The input detection engine pretty much works like this:

<img src='{{ site.url }}/assets/uploads/2015/10/wrong.png'>

The result is that icons never match the input device if you've got a hard-wired gamepad and aren't using it. Another interesting design decision was their mouse timer, even if you are using keyboard/mouse and have the mouse button held down, the mouse cursor will always vanish after 5 seconds of zero mouse travel.

The menu system in GW3 is clearly designed for a gamepad and it does support both analog and dpad for navigation but does not support mouse. The game pauses when the gamepad connection is lost, however it does this even when you've never even touched the gamepad. 

Most of these issues could be resolved just by making the input detection engine a bit smarter.

## Pacman CE DX+ ##
Pacman uses the same input detection scheme as GW3, so icons never match unless you unplug your gamepad. The mouse cursor is never hidden, even with a gamepad detected. In a game where every little twitch matters, it's punishing that it does not pause when the gamepad gets unplugged.

## Tomb Raider ##
The input detection engine doesn't trigger off of mouse movement, it requires a physical button press. The game does not pause when gamepad connection is lost.

## Child of Light ##
This is a good example of input done right!

## Bioshock Infinite ##
Great input detection, they trigger off of mouse movement! Unfortunately no pause on gamepad disconnect.
 
# Summary #
That's all I've got, a high-level input detection engine based on timestamps and 5 simple rules for input. Again, this is targeted at single player games, so keep in mind that if your game supports couch co-op, you'll probably be deliberately breaking a few of these. If you have any questions or comments please leave them below.


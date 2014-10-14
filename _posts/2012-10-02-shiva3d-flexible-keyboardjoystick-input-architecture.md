---
title: ShiVa3D Flexible Keyboard/Joystick Input Architecture
author: error454
excerpt: A new take on input handling in ShiVa.
layout: post
permalink: /2012/10/02/shiva3d-flexible-keyboardjoystick-input-architecture/
publicize_reach:
  - 'a:2:{s:7:"twitter";a:1:{i:304924;i:132;}s:2:"wp";a:1:{i:0;i:45;}}'
categories:
  - Shiva 3D
tags:
  - input
  - joystick
  - keyboard
  - keyboard handler
  - shiva
---
There are a couple different ways to do keyboard and joystick handling in ShiVa.  This article explores a unique method whose merits will be compared against the traditional method.

Here is the traditional way of handling input in ShiVa:

<a href=''><img src='{{ site.url }}/assets/uploads/2012/10/traditionalkeyboardhandling-3.png' alt=''></a>

The pros of this implementation are:

1.  Easy to understand for even the most casual observer
2.  Easy to implement for simple cases
3.  No performance drawbacks

The cons of this implementation are:

1.  State logic is broken up
2.  If your game supports keyboard and joystick, you have to duplicate the state detection code in each handler
3.  Managing multiple AIs that require input can be tiresome

Let's look at these cons more closely.



# Cons of the Traditional Method

## State Logic Broken Up

I love States in ShiVa, they let us cleanly segregate the states of our game.  What I don't like about the traditional implementation is that if you open up a State onLoop function, you don't see the whole picture of what is happening in that state. Instead, you have to open up the keyboard/joystick handlers in addition to the State code to see the whole picture.  Because so much logic is contained in these input handlers, eventually they can become bloated and difficult to maintain.

## Duplicate State Detection Code

Once you add joystick into the mix, you have a situation where duplicate state logic has to be created (am I in the menu state or game state?), one set of logic for keyboard keys and another duplicate set of logic for joystick.

## Multiple AIs

Now imagine you have more than one AI that needs keyboard inputs.  Now you have to forward events from your master AI to the other AIs.  Moreover, you have to duplicate the key input detection code and the state detection in each keyboard handler.

# The Stateful Method of Input Handling

I'm calling this method the Stateful Method, this is what it looks like.

<a href=''><img src='{{ site.url }}/assets/uploads/2012/10/statefulkeyboardhandling-1.png' alt=''></a>

The primary feature of this method is the logic behind the Input AI.  This AI captures button and joystick events and throws them into a hashtable.  It then provides a GetKey method that allows external AIs to query a key state.  I wrote this implementation because I was working on a project that has several disconnected tools and I was tired of managing individual keyboard handlers for all of them.

## Advantages

Some advantages of this implementation are:

1.  The InputAI is abstracted in a way that allows any AI to access it.  No forwarding of input handlers is required.  Every AI now has convenient access to keystates with the minor inconvenience of including a couple helper functions.
2.  Interesting opportunities are now available, like having an object AI read its own input directly instead of being forwarded commands/input.
3.  AI States can now be fully self-contained

Here is what a simple example would look like using the traditional method:



And here is what the new style would look like:



## Disadvantages

There are 2 primary disadvantages to this method.

1.  This method requires that you implement 2 helper functions in each AI that will query input (3 if you want to query latched input).
2.  Input update logic runs every frame rather than being initiated by input.  This could potentially mean performance issues down the road, although I have not noticed any.  It could also mean re-thinking the way you handle input in some AIs.

## Source

The full source to this is available on <a href="https://github.com/error454/ShiVa-Proof-Of-Concept/tree/master/GameShell" target="_blank">github</a>.  InputAI is where the magic happens, getKey, getKeyLatched and getInputAIReturnValue are the 3 required helper functions in GameAI.
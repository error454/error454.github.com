---
title: Obfuscating ShiVa Games with Proguard
author: error454
layout: post
permalink: /2014/01/16/obfuscating-shiva-games-with-proguard/
categories:
  - Android
  - Hacking
  - Shiva 3D
tags:
  - obfuscation
  - proguard
  - shiva
---
<a href='{{ site.url }}/assets/uploads/2014/01/IMG_1724.jpg'><img src='{{ site.url }}/assets/uploads/2014/01/IMG_1724-300x300.jpg' alt='Hiding in plain site'></a>

If you are worried about people hacking your ShiVa based game then obfuscation is a good place to start.  Obfuscation is very simple to implement in your Android project and is a good starting point for anti-hacking measures.
<!--more-->
Nothing will stop a truly determined and experienced hacker, period.  If you rate the skill of hackers from 1-10, you might think of obfuscation as preventing those of skill 5 or less.  When it comes to fighting people that are decompiling your code, you won't win all the time.  Obfuscation is easy to implement and makes a confusing mess of your code, so it's a great low-effort way to prevent the script-kiddies from screwing with your purchase code.

To successfully work with obfuscation, this tutorial will show you:

*   How to modify your build to enable obfuscation
*   What to save after each build to allow de-obfuscating stack dumps
*   How to de-obfuscate stack dumps

# Modify Your Build

By **build **I mean an Android Project UAT build.  In other words, you've gone through the Authoring Tool, built an android project from your STK and have unzipped the resulting zip file to access the build files.

## Step 1  project.properties

Edit project.properties and uncomment the line to read:

<pre>proguard.config=proguard-project.txt</pre>

## Step 2  proguard-project.txt

Replace the contents of proguard-project.txt with:

{%gist error454/8329134 %}

## Step 3  modify line 17 of proguard-project.txt

You need to change line 17 and replace the bracketed variables.  Here is a before/after example showing the replacement of the package and class name.

Before

<pre>-keep class [_PACKAGE].[_CLASSNAME]{</pre>

After

<pre>-keep class com.error454.example.Main{</pre>

Now you can build with ant.

# What to save

Every time you build your app, proguard will generate files under **bin/proguard.  **It is important that you save the **mapping.txt** file so that you can associate it with a specific build of your app.  Once you've uploaded an obfuscated build to the android market, the stack traces that are generated in user reports are going to look like non-sense.  The mapping.txt file will allow you to transform them into the helpful stack traces that you remember.

Remember, every time you do a final build for the app store, save your mapping.txt, preferably as a versioned file in a source repository.

# How to Deobfuscate

The android SDK includes a tool called **retrace **that you'll find under **tools\proguard\bin**.  To deobfuscate your dump, you run:

<pre>retrace mapping.txt obfuscated_dump.txt</pre>

# Conclusion & Caveats

With just a few simple steps, you've obfuscated your game.  Keep in mind that many SDKs have specific changes that need to be added to proguard-project.txt, so be sure to check their instructions for any required additions.  I hope this tutorial was helpful.  If you are interested in some penetration testing against your game, please let me know.

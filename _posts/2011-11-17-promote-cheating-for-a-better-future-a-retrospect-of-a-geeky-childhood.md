---
title: Promote Cheating for a Better Future (a retrospect of a geeky childhood)
author: error454
layout: post
permalink: /2011/11/17/promote-cheating-for-a-better-future-a-retrospect-of-a-geeky-childhood/
tagazine-media:
  - 'a:7:{s:7:"primary";s:61:"http://mobilecoder.files.wordpress.com/2011/10/game_genie.png";s:6:"images";a:1:{s:61:"http://mobilecoder.files.wordpress.com/2011/10/game_genie.png";a:6:{s:8:"file_url";s:61:"http://mobilecoder.files.wordpress.com/2011/10/game_genie.png";s:5:"width";s:3:"298";s:6:"height";s:3:"240";s:4:"type";s:5:"image";s:4:"area";s:5:"71520";s:9:"file_path";s:0:"";}}s:6:"videos";a:0:{}s:11:"image_count";s:1:"1";s:6:"author";s:8:"11758919";s:7:"blog_id";s:8:"11929434";s:9:"mod_stamp";s:19:"2011-11-18 06:13:49";}'
categories:
  - Personal
tags:
  - childhood
  - geek
  - hex editing
  - learning
  - retrospect
  - tutorials
---
<img class="alignleft" title="game_genie" src="{{ site.url }}/assets/uploads/2011/10/game_genie.png" alt="" width="298" height="240" />  
I recently turned 0b100000, on this momentous transition from my 5th to 6th bit, I took some time to reflect back on how I got where I am today.  Of all the events in my life that affected the direction I went, the desire to cheat stands alone as having an overwhelmingly positive impact.



## The Year 1991

In 1991 at the age of 12, I was like any other kid. I spent half my time playing my new Super Nintendo (SNES) and the other half playing with my 486 DX2 running MS DOS 6.X.  Wow, don't even get me started on what a kid gamer had to learn about conventional memory management in the 90s just to run the latest Wing Commander!

On the SNES, one of my friends had a game genie. It was basically this cartridge that you plugged your SNES cartridge into and then plugged the whole thing into the SNES.  The Game Genie came with a booklet that had a list of cheats for all the popular games, stuff like:

<pre>Super Mario: Higher Jump -  D4C7-3FA7</pre>

So you'd plug this monster into the SNES, power on and were greeted with a Game Genie interface where you could enter these codes for your game.  Once you entered all the codes you wanted, you started the game with your cheats enabled.

The cheating was cool and all, but at the time I was more fascinated with how and why this all worked. I mean, think about this a second, you put a number into this black box and on the other end it makes mario jump higher?!?  Whaaaaat?  This was literally a magic genie in a cartridge.

## The Magic of Cheating

I became so fixated on how the game genie worked and the possibility of creating my own codes that I started searching for answers.  The first place I turned was the manual.  I've always had a thing for reading manuals in their entirety and this was no exception.  I found it incredibly dissapointing that the manual didn't tell me how to make my own Game Genie codes.  I even called the Nintendo Hotline, thinking that they would surely know how.  I didn't know the Game Genie wasn't made by Nintendo at the time but I certainly did after the hotline attendant made it sound like the Game Genie was going to cause my SNES to spontaneously combust.

Where now?  Back in 1991, you couldn't hop onto google and start searching for these types of things.  No, back then I had a handful of BBS numbers and AOL.  Not many people may remember, but AOL had access to Newsgroups.  I didn't know what a newsgroup was at the time, I just thought it was like a Prodigy message board but bigger.  I don't recall what search terms I used but I do remember making an incredible discovery that day that changed my life forever.

## Hexediting

I found a tutorial on Hexediting Save Game Files.  The author explained things in a way that a kid my age understood, he didn't know all the terminology or the reasons why things worked the way they did, but he knew enough to get the job done.  At the age of 12, that's really all I cared about.

He started simple, explaining how when you save a file on the computer, it stores things in hexadecimal.  He pointed to some tools to convert from normal numbers to hex (base10 to base16) and mentioned that he wasn't sure why, but sometimes instead of searching for FF AB, you had to search for AB FF.  I had just learned big-endian vs little-endian although I wouldn't know the proper name until years later.  I already had the software I needed (pctools) and with this new knowledge I began an amazing journey.

Cheating was intoxicating, I hex edited everything.  This really wasn't about cheating at all, this was about solving a complex and satisfying puzzle, it was about feeling like I had control of a vastly mysterious universe that was once outside of my understanding.  It gave me the confidence that I could learn anything and it inspired me to dig deeper into how things work.

## I was Legend-

There wasn't any good note taking software back then, and since things were single-tasking in DOS it didn't matter anyway.  I had a notebook full of addresses for all my games.  In Eye of the Beholder 2, I had mapped every character stat, every inventory slot and every item.  Kids at school would bring their saved games on a 3.5 disk and I would call them once I got home and say ok, what sword do you want in inventory slot1?  Hmm, you have a bunch of magic missile scrolls taking up space, is it ok if I delete them?, I was a legend.

I remember the first encrypted file I came across, the game was Shadow of Yserbius on The Sierra Network (TSN).  I used the usual method of saving my game, dropping an item and saving again so that I could compare the files.  I remember that night vividly because the base16 covered over 3 notebook pages, I couldn't make any sense of it, why had so much changed for just 1 item?

Through some social networking, I discovered a BBS on the other side of the US that supposedly contained a document on how to hack this game.  I got permission from my parents, dialed in, chatted with the sysop (system operator) and got the file.  This document was a little above my head at the time but I remember learning that trying to hack an encrypted file was fruitless unless you knew the encryption being used.  Instead this article taught about editing live memory while the program was running by loading a TSR (terminate stay resident aka a background process).  This literally blew my mind and world, I was truly unstoppable at this point.

Once you start editing live memory, you learn what not to do pretty quickly things like trying to promote a single byte value to two bytes (buffer overflow).  A year later with several more tutorials under my belt, I asked for the Turbo Pascal compiler for my 13th birthday (my parents must have thought I was nuts), I actually wanted to program the Z80 (since I knew it was the primary processor in the NES)  but was told I shouldn't jump straight to assembly language.

## Today

I often wonder who the author of that hexediting tutorial was and what he is doing today.  I've done a few searches on <a href="http://www.textfiles.com" target="_blank">textfiles.com</a> but haven't been able to find anything close.  Today, in single player games, I occasionally cheat, it's almost like my own personal crossword puzzle.  I love peering inside save game files to see whether the designers made an attempt at obfuscating the data or whether they just threw their hands up and called it a day.  One of the best tools around is by far <a href="http://cheatengine.org/" target="_blank">Cheat Engine</a>, and not just for cheating in games, I get an amazing amount of utility in my day job (which has nothing to do with video games) with this program.

Cheating was the gateway to a satisfying future and tutorials written by inspired tinkeres were the catalyst.  For anyone who has ever written a tutorial, thank you!  It doesn't take big words or proper names to teach someone a process.  You never know how you might change someone's world by simply explaining how to do something, so hit up <a href="http://stackexchange.com/" target="_blank">stackexchange.com</a>, <a href="http://www.tumblr.com" target="_blank">tumblr</a>, <a href="http://blogspot.com" target="_blank">blogspot </a>or <a href="http://wordpress.com" target="_blank">wordpress </a>and make the world a better place.
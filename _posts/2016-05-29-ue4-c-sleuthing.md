---
title: UE4 C++ Code Sleuthing
author: error454
layout: post
permalink: /2016/05/29/ue4/c/sleuthing/
categories:
  - ue4
  - c++
tags:
  - ue4
  - c++
  - search
  - code
  - github
  - blueprint

---

This is a short article with a few tips on how I hunt down engine code in UE4. There's no big secret here, just a series of tools that can be combined by anyone with a little motivation.

<!--more-->

# Official Docs #

The [UE4 docs](https://docs.unrealengine.com/latest/INT/Search/index.html?x=0&y=0&q=aactor) have a ton of great info, but as a C++ developer I find that the default search is mostly unhelpful. I find it most useful to filter by either Documentation or API.

<img src='{{ site.url }}/assets/uploads/2016/06/ue4search.jpg'>

# Seeing Code for Blueprint Nodes #

You'll often come across some blueprint node that you might want to see the code for. For many blueprints you can right-click and select to see the code, this will open Visual Studio directly to the code of interest for you to inspect.

<img src='{{ site.url }}/assets/uploads/2016/06/viewcode.jpg'>

# Tracking Down Code for BP Properties #

Sometimes you'll want to find code that touches a specific blueprint property. For instance, what the heck does this do? And more importantly how would we go about finding the relevant code?

<img src='{{ site.url }}/assets/uploads/2016/06/findme.jpg'>

First you need to find the variable name for the property. In many cases, you might not even know which class the property belongs to, all you'll have to go off is the property description in the editor. This is when I like to turn to github search. Github search is indexed and lightning fast (unlike searching through VS).

Set a bookmark for the [UE4 github repo](https://github.com/EpicGames/UnrealEngine) now, you'll use it all the time. There are 3 caveats to github search that you should be aware of first:

1. It can't search code in forked repos 
2. It only searches code in the master branch
3. The search results only show the first few matches in any given file

Now let's run through how we'd track down **Actor Hidden in Game**.

## Search on Github ##

Probably the most lucrative way to search is to first see if the code author added any extra comments to this property, this will get you a more unique string to search for. To do this, hover over the property and see what pops up.

<img src='{{ site.url }}/assets/uploads/2016/06/bingo.jpg'>

Cha-ching! Be sure to surround the search with double quotes.

<img src='{{ site.url }}/assets/uploads/2016/06/search1.jpg'>

Well well, only a single result. Click on the line number to zoom in to the area of interest.

<img src='{{ site.url }}/assets/uploads/2016/06/search2.jpg'>

## Digging Deeper in Visual Studio ##

Now that you know the variable belongs to **actor.h** and is named `bHidden` you can dig deeper.

At this point we're done with github search. The problem with continuing the search on github is that it's pretty dumb. For instance if 9 other classes have a variable named `bHidden` it's going to show you all of those matches and it's going to be hard for you to sort them out. So don't believe everything you see without using a better tool that has the context, like Visual Assist. 

To continue, I like to open up Visual Studio solution explorer and drill down to the file.

<img src='{{ site.url }}/assets/uploads/2016/06/search3.jpg'>

I then highlight the variable and use Visual Assist to find all references of the variable.

<img src='{{ site.url }}/assets/uploads/2016/06/vassist.jpg'>

Boom! There are a ton of results, so it depends what we're interested in here. Do we want to know how HUD uses the flag, or if HeightFog uses it or maybe just how Actors themselves might use it? In this case let's see how the Actor code uses the flag. It looks pretty simple, there are only 2 lines where the flag is used. Let's check them out.

<img src='{{ site.url }}/assets/uploads/2016/06/foundyou.jpg'>

Bam! Now you can set a breakpoint etc.

If you haven't used [Visual Assist](http://www.wholetomato.com/) I have to say I felt it was worth the $100 for the personal license. It really helped me keep my sanity with the UE4 codebase.

## Unreal Tournament ##

Finally the last resource I use often is the [Unreal Tournament github repo](https://github.com/EpicGames/UnrealTournament). There aren't any other open source UE4-based games out there that are made by professionals. The UT repo is a gold-mine waiting to be harvested. Not sure how to use a function? Search for it. Want to see how a large project handles their UDamage heirarchy? [Take a peak!](https://github.com/EpicGames/UnrealTournament/search?utf8=%E2%9C%93&q=utdamagetype&type=Code) Bored and want to learn something new? [You don't have to look far](https://github.com/EpicGames/UnrealTournament/blob/e46fadbd476797d4c44d25654e1e4d7291f653a2/UnrealTournament/Source/UnrealTournament/Private/UTCharacterMovement.cpp#L1873).

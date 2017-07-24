---
title: UE4 Blueprint Coding Standard
author: error454
layout: post
published: true
permalink: /2017/06/26/ue4/coding/standard
categories:
  - ue4
  - blueprint
tags:
  - ue4
  - blueprint
  - design
---
I was asked to compile a programming standard for a project. I decided to publish this terse version, it is mostly intended for the blueprint-only programmer.

If you are a C++ programmer, follow the [UE4 coding standard](https://docs.unrealengine.com/latest/INT/Programming/Development/CodingStandard/).

If you are a blueprint programmer, read on.
<!--more-->

# Learn the Fundamentals of Programming

Anyone writing blueprints is a programmer. As a programmer, tools and languages come and go, but the fundamentals stick around and apply to every new tool and language. So I recommend focusing most on learning programming fundamentals, data structures and algorithms because this knowledge will carry over into blueprints. A short list of the highlights that everyone should be familiar with:
	
* Basics
    * Loops
    * Conditionals
	* Data types
	* Pointers
* Object Oriented Programming
    * Classes
    * Inheritance
	* Public/Private/Protected data access
	* Interfaces
	* Encapsulation
	* Separation of Concerns
	* Loose coupling vs tight coupling
* Data Structures & Algorithms
    * Structures
        * Enum
        * Struct
	* Array
	* Set
	* Stack
	* Queue
	* Map
    * Algorithms
        * Sorting
        * Searching
	
You don't need to be a C++ programmer to read and understand the ideas behind these concepts. Grab a book, browse the internet, just learn. These fundamentals will provide you with:

* Tools to solve difficult problems 
* Insight to create cleaner architectures
* Knowledge that will be applicable for any programming language

# General Advice

## Top Down Stepwise Refinement

The best way to flesh out an idea is to use [Top Down Stepwise Refinement](https://en.wikibooks.org/wiki/A-level_Computing/AQA/Problem_Solving,_Programming,_Data_Representation_and_Practical_Exercise/Problem_Solving/Top-down_design_and_Step-wise_refinement). Identify the high-risk items in your design, these are the things that you're unsure of, an API you've never used, a concept you're unfamiliar with. Tackle these first with small prototypes and when they are known, go back to your design to determine how the high-risk item will fit in the overall design.

## 3x Design

When asked to design something, go to the whiteboard or scratch paper first. Sketch out a high-level design. Think carefully about how different actors might communicate with each other. Once your design looks good, move on and think of another way to do the same thing. Do this until you have 3 designs.

The reason for this exercise is because our first idea is rarely our best. By thinking out the problem for a while, we can flesh out these bad ideas so that they can be discarded. By the time the 3rd design is finished, the best path forward should be clear.

## Don't Repeat Yourself (DRY)

If you ever find yourself writing the same piece of code in more than 1 place then you've likely done something terribly wrong! You are missing an opportunity to re-use an existing function/class.

## Write Reusable Code
Imagine that for every piece of functionality you write that you will need to export that 1 thing and bring it into a completely new project. With this in mind, try to keep your dependencies to zero. For instance if you wrote a feature that depended on having a mesh in the level named Meshy McMesh Face, reuse would be tedius.

In the same vein as DRY. Try to think in components. Do you need to give multiple actors the ability to have Hopes & Dreams? Maybe you should make a Hopes & Dreams actor component and then add that component to all actors that need it.

## Naming

Functions should follow a scheme of *Verb Noun*:

* CalculateFallDamage
* UpdateMovementSpeed
* PlayDamageFX
* DestroyHelpers

Variables should follow a scheme of *Thing Property*:

* MeleeDelay
* MeleeDamage
* MeleeSpeed
* AttackMultiplier
	
Booleans should clearly describe their state:

* IsAttacking
* CanAttack
* IsOnCooldown

# UE4 Specific

## Level Blueprint Guidelines

The level blueprint is the single most abused thing in UE4. It's probably easiest to illustrate DRY violations using the Level Blueprint.

**Example**

Say you want to create an elevator that provides a callback to an Actor when it has reached its destination. An implementation might go like this:

* Grab a reference to the elevator in level BP
* Grab a reference to the actor in level BP
* Add an event handler in level BP to call the new functionality from elevator to actor

Ok, now what if we need to implement 10 elevators in 10 other maps? Are we going to copy/paste the same event callback 100 times? It would be better to build this functionality into the elevator by providing a pointer to the thing it should alert. This actor could be registered at design time or dynamically during play (depending on the requirements). 

The general idea here is to encapsulate the functionality of your actor so that it is fully self-contained in the actor blueprint. This means don't build out actor functionality behind event handles in the level blueprint. A good clue that you've done things right is when you can plop your actor into any level and it just works.

## Key Binds

Do not put 1-off key binds in blueprints (unless you intend on deleting it before you commit). Every key input should be mapped and accounted for on the project input page. Check first to see if any existing key binds make sense for your needs. 

Why?

* They are obscure/messy
* Difficult to track down
* By default they consume input, meaning if there is another key event bound to the same key, a 1-off will prevent other key binds from receiving input
* They don't use the binding system

## Functions, Node Size & Readability

Use functions. If you're using Top Down Stepwise Refinement then you should have a pretty good list of function names from your higher level steps. A general rule is that if you have more than 10 nodes in your graph then you should probably reduce some of that logic by using functions. 

You will spend 90% of your time reading your code and 10% of the time writing it, so optimize your code for readability. It's easier to read 3 functions (Update Cooldowns, Update FX, Update Location) than the 30 individual nodes that do all that work. It's also easier to test things when they're broken up into functions because you can easily test the individual pieces of your solution.

## Learn Replication

If you are working on a multiplayer title, you need to understand replication. There may be some small exceptions here, like maybe audio (although it still helps to understand). If you don't understand replication and just start trying to implement features using your single-player knowledge, you will be extremely frustrated because your stuff will appear to work when testing and it will never work in multiplayer.

Someone who understands replication might need to come in and fix your implementation, which might mean throwing away hours of your work so that they can build things with replication in mind. This will probably be frustrating for both of you!

You are ready to write multiplayer blueprints when you understand the following:

* Actor replication
* Variable replication
* Component replication
* Movement replication and what it affects
* Difference between client, server and local proxy
* Net ownership (who owns an actor)
* Net relevancy
* Server, Client and Multicast calls

Start with the Networking content example along with the [official docs](https://docs.unrealengine.com/latest/INT/Gameplay/Networking/index.html).

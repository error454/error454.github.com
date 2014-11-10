---
title: UE4 Platformer Game Analysis
author: error454
layout: post
permalink: /2014/11/10/platformer-game-analysis/
categories:
  - ue4
tags:
  - ue4
  - platformer
  - analysis
  - c++
---

My journal from the disection of the platformer game sample.

## PlatformerGameMode.h ##
* Contains a custom state enum for tracking the states of the game. Also exposes this through a getter but there's no setter, the state gets modified based on the flow of the game.
* Has private variables related to scoring
* Manages the time remaining in the game as well as adding/subtracting from that time based on picking up things in the world
* Has functions that manage saving best checkpoint times
* Pauses/Unpauses the game
* Starts/restarts the game
* Positions the player at the start location

## PlatformerBlueprintLibrary ##
This code is basically a huge set of wrappers to expose functions that live in `PlatformerGameMode` to blueprints. Not all of the exposed functions are a 1:1 mapping. For instance the FinishRace function, notice that there isn't a 1:1 function to call, instead it finishes the round and then asks if the round was won:

    bool UPlatformerBlueprintLibrary::FinishRace(class UObject* WorldContextObject)
    {
    	bool bHasWon = false;
    
    	APlatformerGameMode* MyGame = GetGameFromContextObject(WorldContextObject);
    	if (MyGame)
    	{
    		MyGame->FinishRound();
    		bHasWon = MyGame->IsRoundWon();
    	}
    
    	return bHasWon;
    }

I don't really understand the motivation behind this organization. If I were writing something from scratch, I would have the instinct to expose the Game Mode functions to blueprint and have my blueprints calling the same functions 1:1.

On the other hand, not every function in here is a wrapper for `PlatformerGameMode` functions. There is also stuff that calls through to the HUD, sorts scores and formats time strings. I guess that in a way, it is quite nice to have all of your available blueprints in one place rather than making your designer get a handle to your Game Mode to call Blueprint A or a handle to your HUD to call Blueprint B. Also, you can then treat the blueprints like a traditional getter/setter where you do more careful type checking to make sure inputs and outputs are dealt with.

## PlatformerGameUserSettings ##
This class handles applying and modifying settings in the game:
* Sound volume
* Fullscreen mode

Of interest is the `UPROPERTY(config)` for the sound volume variable. 
https://docs.unrealengine.com/latest/INT/Programming/Basics/ConfigurationFiles/index.html

## PlatformerPlayerMovementComponent ##
https://docs.unrealengine.com/latest/INT/Gameplay/Framework/Pawn/Character/index.html

> The CharacterMovementComponent allows avatars not using rigid body physics to move by walking, running, jumping, flying, falling, and swimming. It is specific to Characters, and cannot be implemented by any other class. Properties that can be set in the CharacterMovementComponent include values for falling and walking friction, speeds for travel through air and water and across land, buoyancy, gravity scale, and the physics forces the Character can exert on Physics objects. The CharacterMovementComponent also includes root motion parameters that come from the animation and are already transformed in world space, ready for use by physics.

The default CharacterMovementComponent is overriden in the `PlatformerCharacter` constructor. The main new things that I see this class doing are:

* Implement sliding - this includes functions to query the sliding state
* Expose a bunch of variables to tune various aspects of the movement mechanics
* Overriding StartFalling PhysWalking and ScaleInputAcceleration

## PlatformerPlayerCameraManager ##
A subclass of `APlayerCameraManager` with an override for:

* UpdateViewTargetInternal

And new functionality for:

* Setting a fixed camera Z offset
* Settings/Getting camera zoom

## PlatformerPlayerController ##
Subclass of [APlayerController](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/Controller/index.html):
> The PlayerController implements functionality for taking the input data from the player and translating that into actions such as movement, using items, firing weapons, etc. These actions are generally passed on to other components of the system; most notably, the Pawn and Camera.

Provides overrides for:

* PostInitializeComponents
    * Creates the in-game menu
* SetupInputComponent
    * Binds the `InGameMenu` action to the public function `OnToggleInGameMenu()`

Special functions:

* Enables click and touch events
* TryStartingGame() exposed to blueprints.
* Toggle in game menu
* Handle to an in-game menu

In the constructor is where the custom Player Camera Manager is set:

`PlayerCameraManagerClass = APlatformerPlayerCameraManager::StaticClass();`

It turns out that UE4 has their own implementation of Smart Pointers, [TSharedPtr, TSharedRef, TWeakPtr](https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/SmartPointerLibrary/index.html). You can see one used for the menu instantiation:

    PlatformerIngameMenu = MakeShareable(new FPlatformerIngameMenu());

## Input Handling ##
The player controller only handles the ESC button to toggle the menu. The other input bindings are handled directly in the pawn.
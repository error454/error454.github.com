---
title: 1,000,000 Stupid Questions for UE4 Developers
author: error454
layout: post
permalink: /2014/10/17/UE4-Crash-Course/
categories:
  - ue4
tags:
  - ue4
  - c++
---

My dumpster of random questions and answers for UE4.

# Logging to the Screen #

    GEngine->AddOnScreenDebugMessage(-1, -1, FColor::Red, FString::SanitizeFloat(Val));
    GEngine->AddOnScreenDebugMessage(-1, -1, FColor::Red, TEXT("YO"));

# UPROPERTY "Missing variable type" #
If you get the error *Missing variable type* on your UPROPERTY line, it's probably because you've put a semicolon at the end of the line.

**WRONG**

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "RobotActor");

**RIGHT**

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "RobotActor")

# How to get a reference to an actor in the level blueprint? #
https://docs.unrealengine.com/latest/INT/Engine/Blueprints/UserGuide/Types/LevelBlueprint/index.html#referencingactors

You can store a blueprint in your class as:

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tether")
	UBlueprint* RopeSegment;

You can spawn an instance of this blueprint:

    AActor* segment = GetWorld()->SpawnActor(RopeSegment->GeneratedClass, &actorLocation, &actorRotation, spawnParams);

# How to get delta time? #
    GetWorld()->GetDeltaSeconds();

# Math functions #
Found in FMath namespace

# How to draw hud? #
* Make a blueprint that inherits from HUD.
* Use the event receive draw hud in the blueprint
* Make sure your GameMode is set to use that HUD blueprint
* Don't try to draw hud from player controller

# How to get the current camera? #

    APlayerController* pc = GetOwningPlayerController();
    AActor *vt = pc->GetViewTarget();
    ACameraActor* camera = Cast<ACameraActor>(vt);
    if (camera) {
        //do stuff
    }

# What is [ACharacter](https://docs.unrealengine.com/latest/INT/API/Runtime/Engine/GameFramework/ACharacter/index.html)? Why are there camera controls in there? #

> Characters are Pawns that have a mesh, collision, and built-in movement logic. They are responsible for all physical interaction between the player or AI and the world, and also implement basic networking and input models. They are designed for a vertically-oriented player representation that can walk, jump, fly, and swim through the world using CharacterMovementComponent.

There are camera controls in there because the example you're looking at (Top Down Camera) has added a camera and a camera boom to the character:

# What is APlayerController? #
> PlayerControllers are used by human players to control Pawns.
> 
> ControlRotation (accessed via GetControlRotation()), determines the aiming orientation of the controlled Pawn.
> 
> In networked games, PlayerControllers exist on the server for every player-controlled pawn, and also on the controlling client's machine. They do NOT exist on a client's machine for pawns controlled by remote players elsewhere on the network.

# What does USpringArmComponent do? #
> This component tried to maintain its children at a fixed distance from the parent, but will retract the children if there is a collision, and spring back when there is no collision.

In the examples, the Camera is attached to the Spring Arm so that the heirarchy is:
Parent: Camera Boom
Child: Camera

# How should I store components in my class? #
Many of the example use TSubobjectPtr to do this. The caveat with this is that TSubobjectPtr can only ever be set in your constructor using the post construct initialize properties pointer

In the .h:

    UPROPERTY()
	TSubobjectPtr<USphereComponent> CollisionComp;

In the .cpp constructor:

    CollisionComp = PCIP.CreateDefaultSubobject<USphereComponent>(this, TEXT("SphereComp"));

# What is a [AGameMode](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/GameMode/index.html)? #
> The AGameMode class defines the game being played, and enforces the game rules. Some of the default functionality in AGameMode includes:
> 
> Any new functions or variables that set game rules should be added in a subclass of the AGameMode class. Anything from what inventory items a player starts with or how many lives are available to time limits and the score needed to end the game belongs to GameMode. A subclass of the AGameMode class may be created for each gametype the game should include. A game may have any number of gametypes, and thus subclasses of the AGameMode class; however, only one gametype may be in use at any given time. A GameMode Actor is instantiated each time a level is initialized for play via the UGameEngine::LoadMap() function. The gametype this Actor defines will be used for the duration of the level.

The Game Mode also sets the default pawn class and the player controller class.

# How do I define an Enum in C++? #
From [the wiki](https://wiki.unrealengine.com/Enums_For_Both_C%2B%2B_and_BP). Create a namespace with the enum in it, usually above your class so that others can see it:

    UENUM(BlueprintType)
    namespace EGameState
    {
        enum Type
        {
        	VE_TitleScreen			UMETA(DisplayName = "TitleScreen"),
        	VE_CharacterCreation	UMETA(DisplayName = "CharacterCreation"),
        	VE_Overworld			UMETA(DisplayName = "Overworld"),
        	VE_Battle				UMETA(DisplayName = "Battle")
        };
    }

    UCLASS(minimalapi)
    class ARPGSimulatorGameMode : public AGameMode
    {
    	GENERATED_UCLASS_BODY()
    
    	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = GameState)
    	TEnumAsByte<EGameState::Type> GameStateEnum;
    };

# Quick Reference for U* #
Properties:

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tether")
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Tether")

Functions:

    UFUNCTION(BlueprintCallable, Category = "Tether")
    UFUNCTION(BlueprintNativeEvent, Category = "Tether")

# Who spawns the default pawn and takes control of it? #
The GameMode does, the interesting stuff is in GameMode.cpp in the function [StartNewPlayer()](https://github.com/EpicGames/UnrealEngine/blob/1d429763ca85bce5a9fc40e7a4848e4a00ac42a8/Engine/Source/Runtime/Engine/Private/GameMode.cpp#L1362)

[RestartPlayer()](https://github.com/EpicGames/UnrealEngine/blob/1d429763ca85bce5a9fc40e7a4848e4a00ac42a8/Engine/Source/Runtime/Engine/Private/GameMode.cpp#L414) is where the magic happens. This code:

* Tries to find a good player start location
* Tries to spawn the player using the configured DefaultPawn 

So if you're trying to prevent the spawn of a DefaultPawn, once simple way is to simply not set the `DefaultPawnClass` variable in your custom Game Mode.

# Is it ok to use flat Classes? #
Yes, but only if you don't plan on revealing their variables/functions to blueprints. All of the UPROPERTY and UFUNCTION stuff requires some special stuff in the header. So it's easier to just create your classes as a sub-object of UObject.

This means that:

    UCharacterContainer* me = new UCharacterContainer();

Becomes

    UCharacterContainer* me = NewObject<UCharacterContainer>();

# Timers #
[From the docs](https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/Timers/index.html).

# How to override hit events in c++? #
Following [this example](https://wiki.unrealengine.com/Reflecting_Projectile_C%2B%2B).

    UCLASS()
    class ROB1E_API AProjectile : public AActor
    {
    	GENERATED_UCLASS_BODY()
    
    	UFUNCTION(BlueprintCallable, Category = "Projectile")
    	void LaunchProjectile(FRotator direction, float impulsePower);
    	
    	UPROPERTY(VisibleDefaultsOnly, Category = Projectile)
    	TSubobjectPtr<USphereComponent> CollisionComp;
    
    	UFUNCTION()
    	void OnHit(AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);
    };
    
    AProjectile::AProjectile(const class FPostConstructInitializeProperties& PCIP)
    	: Super(PCIP)
    {
    	CollisionComp = PCIP.CreateDefaultSubobject<USphereComponent>(this, TEXT("SphereComp"));
    	CollisionComp->InitSphereRadius(0.5f);
    	CollisionComp->BodyInstance.SetCollisionProfileName("Projectile");
    	CollisionComp->OnComponentHit.AddDynamic(this, &AProjectile::OnHit);
    	RootComponent = CollisionComp;
    }

# Beware of Gamepad Axis Events #
They are called every frame even if you have no gamepad connected. If you're sharing variables with other input methods, be sure to guard them:

    void ARob1ePawn::AimGamepadX(float Val)
    {
    	if (UseGamepad)
    	{
    		LastAimX = Val;
    	}
    }

# PhysX #

## Rope Notes ##

My rope is not behaving as expected. If I wrap the rope around an asteroid and try to pull, the simulation goes crazy with the asteroid translation jumping all over the place like vampire bill trying to rescue sookie from a dozen attackers.

I noticed that the Unreal implementation uses PX6DOF joints for all joint types in Unreal. In my own physx implementation in another engine, I created this same simulation but using PXSphericalJoints instead and don't have these types of simulation jitters.

I've tried the usual stuff. Enabling sub-stepping (16 at 0.0013). Increasing position and velocity iterations for my rope segments, asteroid and actor (15,15).

Rope Position Iteration count >= 13 seem to make a big difference.

By disabling rope physics collisions and enabling substepping (5 at 0.00666) with Rope iterations at 13,4 the simulation looks good. Reducing projection to 0.2 seems to make the simulation less likely to explode.

## How to start physx visual debugger? ##
Run `pvd connect/disconnect` in the console
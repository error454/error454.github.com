---
title: Setting up Perforce on Azure for UE4
author: error454
layout: post
permalink: /2014/11/14/azure/perforce/
categories:
  - perforce
  - azure
  - ue4
tags:
  - source control
  - repo
  - perforce
  - ue4
  - azure
  - linux
  - git
---

Today is one of those fun days where I get to wear a few different hats in my independent game development career. We're bringing some interns on and we need a better way of collaborating on projects together.

It's actually amazing that we've hobbled along using duct-tape (git) and bailing wire (dropbox) for so long. This is a little embarrassing, but I'm going to outline what we've been using for the last year.

## The Poor Man's Free Dropbox Git Server ##

* Start with a shared dropbox folder for a project.
* On one end, initialize a git repo
* Use Dropbox selective sync to ignore the .git folder as well as intermediate build folders
    * Note that to do this, you usually have to copy your .git folder out, disable selective sync (which deletes the folder) and then copy the .git folder back in.

<!--more-->
The advantages of this method are:

* With a local git server, the method costs nothing
* You don't have to open your source repo to the world
* You have an extra level of redundancy
* Changes are instantaneous

The downsides are:

* Changes are instantaneous
* 1 person ends up being in charge of all of the revisioning
* Not all applications enjoy files being ripped out from under them and this can create occasional sync issues
* Does not scale to 3 people

## Time to Grow Up ##
I'm a serious person and now it's time to **get serious** about team collaboration. Today I'm setting up a linux server on Azure that will be used to host a Perforce repository for our team that's using Unreal Engine 4. Under the Bizspark program, we get $150.00 of free server credit each month. So it seems silly to pay for hosting without first exhausting this resource.

Azure is pretty darn cool. If you like geek panels, there are plenty to be had:

<img src='{{ site.url }}/assets/uploads/2014/11/azure.jpg' alt='Geek panels in azure'>

## Azure Choices ##
There are a lot of choices when setting up a virtual machine. Here is a record of some of the choices I made. My overall goal was to get the minimum viable server configuration so that all of my free credits would go towards bandwidth and storage. My biggest nightmare is for production to stop because we've exhausted our free credits :o

As an overplanner, I hope that I have greatly over-estimated our usage (fingers crossed).

### Server Tier ###
For server tier, I only need basic, not standard. The higher tier includes load balancing and scaling, not something we need for a code repository shared among 4 people. We're all in the same hemisphere anyways, but for that matter if there was a team member in china, I'd just see if they could tough out the latency ;)

### Virtual Machine Size ###
Perforce does better with more memory, but then again, what doesn't? How much is enough? Their [kb article](http://answers.perforce.com/articles/KB_Article/Recommended-Server-Hardware-Configurations) has the equation:

    Estimated Memory = Number of files * 1.5 KB

Ok, let's estimate this.

### Content Estimation ###
There are a lot of files in a game. You've got static meshes, textures, materials, blueprints, skeletons, animations, sound clips and more. All of these are separate files. If I estimate that our game will contain:

* 1000 unique static meshes 
    * 1/4th of those will contain a skeleton
    * Each has ~ 3 texture maps
    * Each has ~ 2 materials
    * Each has ~ 3 animations
    * Each has ~ 2 blueprints

1000 * (3 + 2 + 3 + 2) + 1000 * 0.25 = 10,250 files

10250 * 1.5 KB = **15 MB**

For the Unreal Engine binary distribution alone we have 13,612 files:

13612 * 1.5 KB = **20 MB**

So all said, 35 MB is all we need to satisfy perforce? Pffff. This is far more modest than I thought. The lowest tier on azure has 768 MB of memory which is about 22X more than what we need. But that's actually the amount of memory allocated to the entire VM, so after the OS is up and running... let's see, Ubuntu Server requires 192 MB these days:
(768-192) / 35 = 16X

I can live with 16X headroom, this allows for slop in my estimate and also unforseen memory usage on the server side. So I'm starting out on the lowest server tier and will monitor our memory usage. If we start getting paged out AND we actually notice it AND it's hindering our productivity then we'll look at bumping up to the next memory tier. But as the original goal is to go spartan, we'll stick with the lowest tier. 

### Storage Replication ###
Geo-Redundant is $5 / 100 GB

Locally Redundant is $4 / 100 GB

An extra dollar per 100 GB gives me peace of mind if my local data center is destroyed. For 500 GB of content, I'll pay an extra $5 dollars per month. If it hits the fan, I know where to find a few extra dollars, for now I'm leaving this at Geo-Redundant.

### Disk Size ###
On Azure, you are billed based on used storage and the threshold is 1 TB. So any unused disk space that is laying around isn't billed. 500 GB sounds plenty for our repo. Our largest project so far has been around 5 GB with all the assets and code.

### Disk Host Caching ###
Going to leave this at None, but honestly didn't spend a lot of time researching it.

## Misc Perforce Config Notes ##
I see that as part of the install perforce package installs, it created a perforce group/user and that it also created a default folder to place repositories:

    Creating home directory `/opt/perforce/servers' ...
    usermod: no changes
    
    Thank you for choosing Perforce. To create and configure a Perforce Server,
    on this host, run:
    
      sudo /opt/perforce/sbin/configure-perforce-server.sh server-name
    
Since I just setup a new disk specifically for perforce, why not just mount this directly to /opt/perforce/servers and be done. After fdisk, mke2fs, and editing fstab we're ready to rock. The first time I mount, I notice that it changes permissions on the servers folder to root:root 755, so we need to chown back to perforce:perforce and chmod back to 700.

A quick reboot test to see if things worked:

    # df
    Filesystem     1K-blocks    Used Available Use% Mounted on
    /dev/sda1       30202916 1094896  27841988   4% /
    none                   4       0         4   0% /sys/fs/cgroup
    udev              338020       8    338012   1% /dev
    tmpfs              68644     344     68300   1% /run
    none                5120       0      5120   0% /run/lock
    none              343220       0    343220   0% /run/shm
    none              102400       0    102400   0% /run/user
    /dev/sdb1       20509308   45000  19399452   1% /mnt
    /dev/sdc1      515929528   71448 510598828   1% /opt/perforce/servers

    # ls -l /opt/perforce/
    total 16
    drwxr-xr-x 2 root     root     4096 Nov 14 11:33 bin
    drwxr-xr-x 2 root     root     4096 Nov 14 11:33 sbin
    drwx------ 3 perforce perforce 4096 Nov 14 11:42 servers
    drwxr-xr-x 3 root     root     4096 Nov 14 11:33 usr

Cha-Ching!

### Ports ###
I've setup perforce to run using SSL. I've also required passwords for users, prevented automatic user creation, created some users and set their passwords. I've decided to start by grouping users into 2 groups:

* artists
* developers

I then set depot permissions based on group. Future note, this is how you create or administer groups:

    p4 group groupname

Depots are created using:

    p4 depot depotname

Protection is configured using protect:

    p4 protect

## Unreal Engine 4 Organization & Workflows ##
My end goal is to host two depots in perforce:

1. A binary engine distribution built from latest stable git release
2. Our current work in progress.

### Artist Workflow ###
For a new artist, I want the workflow to be:

1. Sync perforce depots
2. Run UnrealVersionSelector to register engine directory and setup shell extensions
3. Double click .uproject file and they're off to the races

Artists will then use the built-in perforce support to manage source controlling assets. When a new version of the engine is compiled, they'll need to repeat the new artist workflow. My primary concern here is that artists will forget how to use P4V to sync the engine because they'll do it so infrequently.

### Developer Workflow ###
Developers will use git to manage the C++ side of the project and will push binary versions of the engine and project to perforce. I feel deceptive using the word *developers* plural because I'm the only developer.
 
Here is how the repos overlap for developers i.e. me. Note that project name is **Rob1E**.

<img src='{{ site.url }}/assets/uploads/2014/11/repos.jpg' alt='Whiteboard shot of my repo layout'>

The crossed out items are things that are ignored in both repos. You can see that git would ignore all perforce hosted folders in addition to the crossed out items and vice versa for perforce.

To share the compiled pieces of a game project, I simply compile the **development** solution and then push the game dll/pdb file found in the Binaries/Win64 folder to perforce.

### Why use 2 repos? ###
Why am I using 2 repos? Why not just go all-in on perforce? Or conversely all-in on git?

The last question is easy. Git would be a difficult workflow for artists. I want to spend my day coding games, not solving merge conflicts.

Why not go all in on perforce? My primary reasons are:

* I like the git workflow
* My git repo disaster recovery plan is tested and proven
* I don't feel like the artists need access to the code, not even read access
* I don't want to lose my commit history

I'm sure some of these issues are surmountable and I'd love to be convinced of a better solution by someone. If I ever get to the point where I spend more time managing the system than I am using it, then it's simple enough to dump my code into perforce. 

One potential downside that I foresee is that version tagging might get out of sync between the code and asset repos. But time will tell.

## That's It! ##
Time to take my IT hat off and put my gamedev hat back on :)
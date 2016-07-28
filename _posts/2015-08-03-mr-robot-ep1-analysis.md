---
title: Mr. Robot Episode 1 Analysis
author: error454
layout: post
permalink: /2015/08/03/mr/robot/episode/1/analysis/
categories:
  - tv
tags:
  - tv
  - mr robot
  - misc
  - analysis
  - astsu
  - astu
  - elpscrk
  - chmod
---

Mr. Robot was recommended to me and I finally got around to checking out the pilot episode. I dig the show, I really dig that they made an effort to use real stuff for the computer scenes. As I was watching the screens fly by I was thinking "wow, I think they got it right, this is the real deal!".

I thought it would be interesting to freeze frame some of these scenes and take a closer look at what’s going on, will I come away more or less impressed? Let's find out! But before I launch into this I want to say that I know how difficult it is making creative content and entertainment. Regardless of what I find, I'll thumbs up the content creators as I think they did a good enough job pulling the wool over my eyes in the heat of the moment.

<!--more-->

# Frame A #
<img src='{{ site.url }}/assets/uploads/2015/08/robot-00001.jpg'>

## `root@elliot` ##

I’m about to read a lot into this little prefix: The format for this prefix is `username@machine_name`.  

First off, he’s running as root… this could be a clue that he’s a fast and loose kind of guy that prefers root to sudo. Does he really need to be root? Perhaps! You see, we know that Elliot is running ping to find some machine on the network, it’s possible that he regularly uses various ping switches that would require root:

* Flood ping
* Wait interval < 0.2 seconds
* Preload > 3

We also see that he named his computer after himself, interesting! Maybe this could be an insight into his frequent conversations with himself, perhaps the voice in his head is personified by his computer. Setting your machine name to your own first name seems a little kindergarten at first glance, especially for a bright security hacker, but I could be giving too much street cred to cool and arguably less obvious computer names.

I can imagine that if I asked Elliot what’s up with his lame computer name he’d probably reply that if someone got his computer, it’s already too late, it doesn’t matter what the name is.

## The Prefix Problem ##
Honestly, the ping command output is where I start to notice things falling apart a bit. Ping is used to locate machines on a network if you didn’t know already. Here’s what an actual ping command looks like on linux or bsd:

    root@computer:~$ ping -c 1 127.0.0.1
    PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
    64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.087 ms
    
    --- 127.0.0.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.087/0.087/0.087/0.000 ms
    root@computer:~$

What’s the difference? In the screenshot above, they’re replicating the user prompt `root@elliot` onto every line of output. On normal boxes, the user prompt is not displayed when a program prints output. I don’t know of a way to replicate this output without piping every command to some other command, I also don’t think most people would want their prompt appended to their output.

Please tell me if you’ve seen this in the real world, I’d really like to know!

## Ping ##
Most people run ping and then hit Ctrl + C to stop it once they’re satisfied, however since we never see a `^C` appear on Elliot’s prompt, we must assume that he is invoking ping directly with the parameter `-n 1`. Also, the ping statistics section is totally missing and there isn’t an invocation I could find that would squelch that output. 

Perhaps Elliot was just sick and tired of `ping -q` (supposedly the quiet mode) spitting out a bunch of extra lines and so one night while rolling on his pain-killers he compiled a personal version of ping with the brevity he desired.

Moving beyond the issues, we see that Elliot has found the machine he’s looking for, what’s next?

## `elprscrk` ##
My guess is that this fictional program stands for **E**lliot’s **L**eet **P**a**S**sword **CR**ac**K**er. The first invocation is cut off but reads:

    elpscrk -list pswList.list-add Dylan; June 3rd, Stonehenge

The intent seems clear, Elliot is trying to crack a password and is adding some dates and words to help his dictionary attack. Here’s my problem with this, the semicolon… in linux, the semicolon is literally the end of command character, so this command would be split into two commands:

    elpscrk -list pswList.list-add Dylan
    June 3rd, Stonehenge

Last I checked there is no program called June :/

Moving on, I’m guessing that the the list count of 9,875,894 is the number of permutations that will be attempted. It seems like a small number considering the password length is 10 characters. It doesn’t say Permutation Count, so it *could be* that each list is a unique dictionary by itself… that would be a lot of dictionaries! 

The 2nd invocation of `elpscrk` has Elliot specifying the IP address of the machine that he pinged earlier. He’s also specifying what looks like a user `mich05654`. What machine is this exactly? Is he trying to crack the password by attempting to login to this server with user `mich05654`? Typically there are timeouts that make this difficult. Maybe this is a server at his work and maybe the password he’s hacking is his co-workers. If so, interesting naming convention you got for your employees! Hmm, this will remain a mystery.

# Frame B #
<img src='{{ site.url }}/assets/uploads/2015/08/robot-00006.jpg'>

## `Map sectionus34567` ##
This is a fictional command and also highly suspicious as it uses a capital M. It’s really unclear what it would do. One guess is that it’s a command that loads a specific set of environment variables, similar to something like:

    source build/envsetup.sh

Heck let’s go with it. The M is probably capital so that it doesn’t conflict with the [other map command](http://www.tutorialspoint.com/unix_commands/map.htm) that maps to and from unicode. The parameter `sectionus34567` is the section of the data center that we’re interested in… I’ll admit that `sectionus34567` is a very unwieldy name (better would be han or chewy). But once decoded, it makes perfect sense:

**Section US, Row 34, Column 56, Cage 7**

That’s one hell of a big data center, we would have to infer from the above that there are 10 cages per grid unit.

## `Locate server WBKUW300PS345672` ##
Again with the capital letter commands! Clearly this is so the Locate command doesn’t interfere with the existing lowercase locate command (the one that helps you find files). I won’t try to decode the first part of this machine naming scheme other than to say that it appears the server location is also reflected in his name, with 1 additional detail. 345672 CLEARLY indicates **Row 34, Column 56, Cage 7, Blade Chassis 2**.

What’s interesting is that after running the locate command, the command prompt changes to indicate that Elliot is now either connected to machine `WBKUW300PS345672` or perhaps has more environment variables setup so that future alias’s will use this machine. It’s entirely possible/probable that the data center is using key-based passwordless login to ssh between various local machines. So the Locate command could have fired of an ssh in the background and you know, suppressed a bunch of the normal login output.

## `astsu  - info -backup -short` ##
The `astsu` command is another fictional command that had me puzzled for awhile. The space between the first dash and the word info is what made me pause. I thought at first that it could be a custom sudo command, similar to doing `sudo su -`. But there are other invocations where they do `astsu -close` so I had to throw that theory away.

The author did some pretty amazing parsing of input parameters. My guess is that this is another utility that Elliot wrote. The command probably stands for **A** **S**erver s**T**atu**S** **U**tility. The commands seem pretty obvious so far:

* info - **gives you info on the server**
* backup - **prints the backup server**
* short - **keeps the details short**

The output isn’t anything special. It’s an interesting design decision to force the user to specify the full `-backup` (probably `-b` or `--backup` would be more realistic). The more I think about this the more it seems out of character. Here’s a programmer that has spent an absurdly huge amount of time parsing input parameters to allow for spaces between dashes and a disproportional amount of time on the output layer where they abbreviate `backup` to `bkup`. This is highly suspect when the words `server` and `backup` are both 6 characters long and would align nicely in a printed output with a fixed width font.

Let’s be honest, so far Elliot seems like a lazy coder with little to no stylish sensibility, if I had to guess, he wrote the input layer while high and the output layer the day after his anti-addiction drugs ran out.

# Frame C #
<img src='{{ site.url }}/assets/uploads/2015/08/robot-00008.jpg'>

This frame has a few different invocations of `astsu`.

## `astsu -close port: * -persistent` ##
We really see the power of `astsu` here as Elliot shuts down all the ports on the server. I wonder if `astsu` just turns around and uses iptables?

## `astsu - ifconfig - disable` ##
More magical spaces! At least `ifconfig` is something that we’re all familiar with. Here it looks like Elliot is disabling the ethernet interfaces. It makes you wonder why he closed the ports first if he was just going to yank the whole rug out from under things. This could be a best practice as there’s a slight hiccup bringing interfaces back up later on.

## `Locate BKUW300PS345672` ##
Here Elliot switches to the backup server. He runs his usual `astsu` wank to get details on the server. This one comes back as offline and has no default gateway! It also has a domain controller…

## Ethernet Wrangling
`set waneth0* : * 23.234.45.1 255.255.255.0 [45.85.123.10; 45.85.124.10; 45.85.125.10]`

This command appears to be bringing up all of the ethernet devices that were brought down earlier. Elliot is specifying the default gateway explicitly `23.234.45.1` as well as the 3 DNS servers in square brackets.

All of the ethernet interfaces come up except for `waneth04` which says *failed*, so Elliot man handles it:

`set -force -ovr02 waneth04 : 23.234.45.62:441 23.234.45.1 255.255.255.0 [45.85.123.10; 45.85.124.10;`

Right, the good ole `-force -ovr02` trick, seems to have worked. Tuck that one in your pocket sys admins.

## `astsu -open port: * -persistent` ##
Elliot opens up all the ports.

# Frame D #
<img src='{{ site.url }}/assets/uploads/2015/08/robot-00013.jpg'>

## `ps aux|grep root` ##
Hey hey! This looks legit…. at first… except that there should be about several dozen other processes on a normally running system. I’m gonna let it slide and move right to the bigger issue here, is Elliot really an `aux` type of guy instead of an `-ef` type of guy? Can we infer from this that Elliot cut his teeth on the BSD side of things rather than POSIX? Hmm, deep thoughts here, very deep thoughts.

This output is interesting, clearly we see the `evilscorpwb*` processes, those probably shouldn’t be running. But what’s going on with PID 24? `cpuset ./01dat`. Also to note is the `-20` on the command line, I’m guessing this is a nice level since those range from -20 to 20. What nice does is change the priority of your process. So here we see that someone has given their process maximum priority! Also `cpuset` is a utility that allows you to assign an application to a specific set of CPUs. This *could* be the intent here but it’s hard to say.

## `astu trace -pid 244 -cmd` ##
Lots of issues here. First of all, what the heck is `astu`, did Elliot mean to type `astsu`? Does Elliot have yet another custom program called `astu`? Wow, thanks for the great naming convention Elliot. Furthermore, there is no pid 244 in the output, does he mean pid 24 (the suspiciously niced `cpuset `command).

Presumably what Elliot meant to type was:

    astsu trace -pid 24 -cmd

`astsu `already does everything else, may as well add an `strace` like feature too. We see an inconspicuous `trace placed` message after this and presumably Elliot watches the fopen calls fly by to find where on the filesystem this process is reading/writing.

# Frame E #
<img src='{{ site.url }}/assets/uploads/2015/08/robot-00014.jpg'>

## `ps aux| grep root|cpuset` ##
Well, looks like `cpuset` is another piece of custom work, because you can’t just pipe stuff to it. Perhaps the intent was to also grep for `cpuset`. Regardless, a process was returned with PID 4, it has a nice level of -20 so I’m guessing this is the `cpuset` we saw in Frame D.

## `astu -ls ./root/fsociety/ -a` ##
Presumably a typo that was meant to be `astsu`. Here Elliot lists the contents of the `root/fsociety` directory.

# Frame F #
<img src='{{ site.url }}/assets/uploads/2015/08/robot-00016.jpg'>

The file listing comes back followed by arguably the single most disappointing command in this entire series.

## `more readme.txt` ##
Elliot uses `more`? Whaaaaat? I would have pegged him for a `less` kind of guy. Let’s talk about the contents of readme.txt. This was Elliot’s immediate reaction.

<img src='{{ site.url }}/assets/uploads/2015/08/robot-00019.jpg'>

His reaction was not to the message itself, but to the sloppy asymmetric header that was missing a dash on the left side. Was this intentional? Did they know that this would bother Elliot?

# Frame G #
<img src='{{ site.url }}/assets/uploads/2015/08/robot-00018.jpg'>

## `sudo kill 4` ##
Elliot kills the `setcpu` process that was first PID 24, typed as PID 244 and then mysteriously changed to PID 4.

## `astu -rm -norecycle /root/ fsociety/` ##
Whoah whoah whoah there tiger. First of all, the dreaded `astu` command returns, we may need to refactor our entire theory on `astsu` and its many functions. The willy nilly Elliot is deleting some files and skipping the recycle bin. Unfortunately he didn’t escape his space and is lucky he canceled the delete otherwise he would have blown away his entire root directory.

## `chmod -R ER280652 600` ##
Elliot does a recursive change mod to presumably change the permissions of the fsociety folder so that only the owner can read/write it. The `ER280652` is presumably a user name or group. Unfortunately `chmod` doesn’t let you specify user/group, you use `chown` for that and `chown` doesn’t let you specify a new permission, you use `chmod `for that. This is a combination of both, so I’m guessing that Elliot wrote his own `chmod`. Busy guy!

# Conclusion #
This was fun, but the ping-ponging between `astsu` and `astu`
 has me a bit bewildered. I'd like to see someone do an in-depth analysis of the two tools and what their individual functions are.

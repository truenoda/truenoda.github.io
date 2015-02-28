---
layout: post
title:  "Wizard of Bits, Systemd? More like Shit-stemd"
date:   2014-10-9
author: wizardofbits
categories: 
- blog
- Linux流
thumb: thumb06.png
---


Wizard of Bits, Systemd? More like Shit-stemd

<!--more-->

Systemd is a critical component of many Linux systems. I fucking hate it.
I’ll get into why, but first a few definitions:



-  A program is a sequence of instructions for a computer.
-  A process is a particular instance of a program while it’s running. Processes are, approximately, to programs what flights are to airliners. In many systems, processes are given numbers, called process IDs or PIDs.
-   A daemon is a program that is usually started automatically, and runs continuously. It does not interact directly with the user, but waits for something to happen and then acts on that event. For example, it may be waiting for a specific time to perform a scheduled task, a network connection to be made, or a file or message to be written somewhere specific. They often handle system maintenance, cleanup, or server tasks.

<br><br>

When you boot a Unix or Unix-like system, the very first program that gets loaded and run is called “init”. It stands for initialization; and, true to this name, its purpose is to start running all the other programs needed to get the system running so people can log into it. In fact, hardcoded into the kernel is a directive to look for a program called “/sbin/init” and run it, shutting down when that program finishes its run.

<br><br>

Init is also the last process to terminate when the system shuts down, and is responsible for notifying the other processes that it’s shutdown time, and forcibly terminating any stragglers. When init itself terminates, the whole system terminates. THIS IS IMPORTANT.
Init has one other job, which is to keep the process tables clean. See, any process can create a copy of itself (called “forking” (don’t laugh) in Unix terminology); this is usually a precursor to loading some other program. Any process that runs to completion can deliver a status code to the process that created it; the creating (or parent) process is said to “wait” on the status code of the created (or child) process. But what happens if the parent process dies before the child does? What happens is that init is designated to be the adoptive parent of the “orphaned” process, and it waits for, and discards, any status code that may be returned by the orphan on exit. This is to prevent “zombie processes” – process table slots that are filled with status codes but have no running programs attached to them. They are undesirable because they take up process table space that could be used by actual, running programs.

<br><br>

So it is important that init run well and not crash.

<br><br>

Now, in Unix system design, it is a generally understood principle that a big task not be handled by a big program, but rather a collection of small programs, each tackling one specific, well-defined component of the larger task. You often hear the phrase “do one thing, and do it well” as a guiding principle for writing a Unix program. One major reason for this is that a small program has fewer places for bugs to hide than a big program does.
Until recently, the traditional init program for Linux was called “sysvinit”, because it is supposed to be compatible with the init system from AT&T Unix System V. Usually it uses a complex series of shell scripts to bring the system up. The major alternative to sysvinit was called BSD init, which runs one shell script to bring the system up. Some Linux systems deployed sysvinit “BSD-style”, meaning that instead of the complicated tangle of scripts, there was only one, just like on BSD systems. The single shell script in a BSD-style init often incorporated other sub-scripts that set various configuration parameters; but even with these it was vastly simpler than the sysv scripts were.

<br><br>

So most deployments of sysvinit were, frankly, a mess. Enter Canonical, Ltd., distributors of the popular Ubuntu linux system. Rather than use a BSD-style startup process, or attempt to bring any sort of sanity to the init procedure, what they decided to do was replace init altogether.

<br><br>

Replacing init is fun, and something any experienced Unix nerd should try doing. Writing your own system utilities, including init, can be enlightening when it comes to finding out how a system works at a fundamental level. If you want to get wacky, you could even – say – boot with emacs as your init and boot straight into your editor. Such a system would probably not be stable so it’s not recommended for long-term use, but it’s fun to try on a lark.

<br><br>

However, another Unix system design principle is, if it ain’t broke, don’t fix it. For production systems you really want to leave well enough alone; and the sysvinit that comes with Linux is simple enough, well-understood enough, and stress-tested enough that you really don’t want to mess with it to get actual work done.

<br><br>

But that principle never stopped Canonical. They just love to mess with shit over there. Upstart is a mess. Instead of a script it has a clusterfuck of config files. Instead of simply running a program to start the daemons it needs to start, it has what’s called “event-based startup”, in which every started or stopped program fires off an event, which the config files map to actions – usually starting or stopping other programs. It’s very sophisticated and difficult to manage.

<br><br>

Some workloads – heavy duty application servers, for example – have really sophisticated process management needs. Processes can fall over at any time, and you need other processes to monitor the work processes and restart them as necessary if they fail. If any process depends on the failed process, it too may need to be stopped and so on. It’s perfectly legitimate to write a monitor program that tracks all these dependencies and takes appropriate action. BUT IT SHOULD NOT. REPLACE. INIT. Write the program, make it subordinate to init, and then add it to init’s startup script. Init’s job is to serve as the primordial process, starting all the other processes needed to get a functioning system, and then lock up at the end of the night. If you add all this other crap to init – all this complexity and cleverness – and some part of that breaks, causing init to dump core or something, guess what you have? An unstable system. Remember, if init dies, everything dies.

<br><br>

Init should be kept simple and stupid. There is NO reason for it to be otherwise.
So that’s Upstart, which was bad enough. Now we get to systemd, which takes all of the bad stuff I mentioned above and adds even more ways to do it wrong. Systemd was originally written by a guy with the Rowling-esque name of Lennart Poettering for Red Hat, and boy is it a hot mess. See, the problem with systemd is that it’s not just an init replacement; if there’s a daemon running on a Linux system, systemd probably wants to replace it. It’s replaced udevd, the device node manager. It’s replaced inetd, the internet service super-server. Recently it also gained the functionality of cron, the daemon which runs scheduled tasks at the appointed times. It has its own logging, and since systemd logging is no longer in text files, its own HTTP server so you can read the logs. It’s replaced ConsoleKit. I don’t even know what the fuck ConsoleKit is supposed to do, but desktop environments use it. If your desktop setup depended on ConsoleKit, it now depends on systemd.

<br><br>

Some of these are not in pid 1 but in other little daemons that are tightly coupled to systemd. But pid 1 already does too fucking much. It configures network interfaces, mounts file systems, sets the hostname, opens and listens on sockets to start daemons on-demand, runs cron jobs, builds and checks and fiddles with a dependency graph to determine which daemons start when, and so on. It depends on d-bus for Christ’s sakes – and exposes a d-bus interface at runtime. All this SHIT living in pid 1, plus the fact that you have all these tightly-coupled daemons makes me, an old-time Unix-head, throw up in my throat a little. THIS IS NOT HOW YOU UNIX. Just because you put different things into different .c source files doesn’t mean you’re modular, or decoupled, or you’ve followed the Unix way. Init should depend on VERY LITTLE and do VERY LITTLE. Simply because IT’S THE FIRST THING TO START UP AND THE LAST THING TO SHUT DOWN. Init should get the system up when almost nothing else (save the kernel and libc) is available, and it should STAY up for as long as the system is expected to run (which could be years without a shutdown or reboot!). The more libraries and shit you depend on, the more libraries and shit you need upon system startup (which you may not be able to afford if you have a tiny boot volume or initramfs). The more cleverness you have in the init binary itself, the greater the chance that there could be an init-crashing (and hence system-crashing) bug in there. I don’t care how smart you are. MORE CODE = STATISTICALLY MORE BUGS. LESS CODE = STATISTICALLY LESS BUGS. I will put my trust in the init that has less code EVERY. SINGLE. TIME.

<br><br>

The upshot of all this is the following:
**SYSTEMD IS THE EXACT OPPOSITE OF WHAT A GOOD INIT SHOULD BE.**
In fact, you know what it feels like? An invasion. It feels like we woke up one day and the entire infrastructure of our Linux systems was replaced with the whims of a couple of developers (Poettering and Kay Sievers). This is actually the case if you run bleeding-edge Fedora or Arch; as you got systemd with one of your automatic package upgrades. Lennart Poettering did not engage with the rest of the Linux community, and he did not appear to really grok fundamental Unix system principles before setting out to write systemd. He just decided that the critical bits of everybody’s Linux system needed to be replaced with his code. Why? Because fuck you, that’s why. There’s a lot of hate for him on the internet, and a lot of it is just unwarranted flamage. He’s a fairly clever guy, and systemd is pretty stable for all its sophistication. But if you break Unix principles, and then try to go over the community’s head to get your tightly coupled monster pushed on everybody, expect there to be some backlash.

<br><br>

Lennart posted a blog called The 30 biggest myths about systemd in which he addresses some of these concerns. It’s a fair attempt at a rebuttal but unfortuantely Lennart, YOU LOSE. YOU LOST WHEN YOU DECIDED THAT PID 1 SHOULD DO A LOT.

<br><br>

Is sysvinit a shitfest? Unless you deploy it BSD-style, yes. Is systemd better in some respects? Yes. Systemd does confer some advantages – allowing for sophisticated process monitoring and control, daemon startup in parallel for fast boot times, etc.
First of all – no one running a desktop or even a laptop Linux workstation really gives a shit about boot times. Most people either leave their machines on or suspend them when not in use, meaning that in most cases, whether the machine starts up in 5 seconds or 30 is a micro-optimization. Especially when compared with the several minutes it took a Windows box to grind to life until very, very recently; and even then, I think Windows was helped more by the proliferation of solid-state drives than a tightly bummed boot process. For servers, boot time is even less of an issue; it takes longer to initialize all the super-special RAID cards and networking hardware than it does to boot the OS.
(Fun fact, I once had to optimize the boot time of an embedded Linux robot platform. One of the options I considered was replacing init – with a tiny binary that ONLY started the requisite services for that platform. I was trying to solve the same problem Lennart was, and STILL came up with the opposite conclusions he did for systemd.)
My Slackware machines running sysvinit take about 30 seconds to boot; most of that time is spent by the kernel probing for hardware. The init boot process takes about four seconds or so. It helps that Slackware’s init scripts are BSD style. My systemd-running work box running Arch doesn’t take an appreciably shorter amount of time.
Secondly, no one BUT NO ONE gives a shit about your shell being a low pid when you first log in. It doesn’t. Fucking. Matter.

<br><br>

But even if you wanted these advantages, there are ways to get them without FUCKING BREAKING UNIX. There’s an init system called runit which offers many of the benefits of systemd, but is built in a much more modular, Unix-like fashion.

<br><br>

Runit allows for sophisticated process control, including starting, stopping, monitoring, and restarting services; and special handling in case the service crashes. It can start services in parallel. It uses a directory full of scripts in order to control startup, shutdown, and error handling of a daemon; and it uses named pipes instead of D-Bus to expose its process control mechanism to user space. Service dependencies are as simple as starting the depended-on service in the run script of the depending service.

<br><br>

This simplicity is what makes Unix beautiful, and it makes runit harmonious with the existing Unix ecosystem. Runit runs on Linux and BSD, and is available as a package for varieties of both. It can be run as pid 1 or you can skip using runit’s init daemon, and gain all the benefits of runit by using the rest of it with your own init daemon. In particular, runit integrates well with sysvinit. ‘sv’, the runit service starter, can simply be symlinked under /etc/init.d under a different name, and if it detects it’s being started this way it will behave as a sysvinit script to start the runit service of that name.

<br><br>

My personal boxen don’t use runit. A BSD-style init script is enough for them. But if I were running a server farm? I’d give runit a long hard look. But I abandoned Arch, a deservedly popular and well-respected Linux distribution, because Arch switched to systemd (and also because of Arch’s tendency to break everything at inopportune monments). I switched back to Slackware, which is still using the same simple, elegant BSD-style init setup that it was in 1995, when I first started using it. And, to be honest, I couldn’t be happier. My life is easier for it.

<br><br>

In short, fuck the hostile takeover that is systemd. Linux is, first and foremost, a Unix; and I will fight to preserve the legacy of simplicity and elegance that entails. At least on the machines I own. You can run whatever the fuck you want; it is, after all, a free country… and it is free software.




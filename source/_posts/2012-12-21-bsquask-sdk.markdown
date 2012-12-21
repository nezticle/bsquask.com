---
layout: post
title: "Bsquask SDK"
date: 2012-12-21 17:39
comments: true
categories: 
---

If you're visiting this page, it likely that you are familiar with my project [RaspberryPi-Buildroot](http://github.com/nezticle/RaspberryPi-BuildRoot) (aka the Bsquask SDK).  If not then, the please check it out!

The objective of this project is to provide an SDK and root file system for the Raspberry Pi that is lightweight and takes full advantage of the hardware available. The resulting image produced is small Linux distribution known as Bsquask.   

The Bsquask SDK provides a GCC 4.6.3 toolchain for building armv6 binaries with the hard-float ABI, as well as bootloaders, kernel image, rootfs, and development sysroot for the Raspberry Pi.

I created the Bsquask SDK because I needed an easy way to work with Qt 5 on the Raspberry Pi.  In general when you are working with an embedded device, you need 3 things: a toolchain, a sysroot, and a system image.

### Toolchains   
When I first started working with the Raspberry Pi in May of 2012, I began a search for a Linux distribution that I could base my Qt 5 development work on top of.  I use Ubuntu Linux as my host OS, so I'm most comfortable with Debian based distributions.  So naturally I was interested in the [Raspbian](http://www.raspbian.org) project.  It was then that I was surprised to learn that to build the Raspbian distribution they were using i.MX53's to build packages natively for armv6 hard-float.  They use rigs that looks sort of like this:   
![buildfarm](http://www.einval.com/~steve/photos/armhf-minirack.jpg)   
The crazy thing though is that it took months to build the base Debian packages.  Thats right months!  On my Quad-core desktop it can take between 20-60 minutes to build Qt 5 (depending on which modules I build), so I can only imagine how long that would take doing native builds.  

**Just say no to native builds!**   

The correct way to develop for embedded devices is cross compilation.  That is having a toolchain that is a native binary for your host machine (x86_64 Ubuntu 12.04 in my case), and generates binaries for your device (armv6hf Raspberry Pi).  This means that builds for your device take roughly the same time as they do for your desktop.  So rather than taking a month to build a base Linux OS, which you may need to rebuild if you want to change a base-layer package, you bring the time scale back down into the order of minutes.  In the case of the Bsquask SDK, we cross compile every dependency of the base OS and that takes between 1-2 hours.

For the Bsquask SDK, I actually generated the toolchain using [crosstool-NG](http://crosstool-ng.org/)

### Sysroot   
When you are developing middle-ware for an embedded device having a complete sysroot is very important.  The sysroot contains all of the development files needed to build software for a given OS image running on your device.  So this includes the development headers for libraries, and the cross-compiled libraries to link against.  For the Bsquask SDK I chose to use [BuildRoot](http://buildroot.uclibc.org) to generate the sysroot and image.   
![buildroot](http://buildroot.uclibc.org/images/logo_small.png)   
The way this works is that I define a recipe on how to cross compile an entire image for a given device.  BuildRoot downloads the source code of each package, compiles it using our toolchain, then installs the resulting binaries, headers, libraries into a sysroot.  The recipe also specifies which files from each project need to go into the image that we will deploy to the device.  So on the device we have limited space, and no need for the development header files, so we do not copy those to the device target folder.  When we build the entire system specified in the recipe, and a kernel, these are then compressed to tar.gz files that become the system image.

### Images   
So what we get in the end is two tar.gz files that we can extract to the two partitions of a Raspberry Pi compatible SD card.  You boot up the Raspberry Pi with this SD card, and now you have a device that you can easily build new software for.  In the case of the Bsquask SDK you just boot up to a login prompt (no X11).  The recipe builds the minimum packages necessary to use all the functionality of Qt 5, which should fulfill all of your embedded device middle-ware needs (and if it doesn't thats something I should know about).

### Conclusion   
The Raspberry Pi is an awesome device, and a blank slate for many developers creativity.  In general embedded Linux development is difficult, but by combining the Raspberry Pi ease of use (just throw all files on the SD card), and the complete development environment provided by the Bsquask SDK, you are only limited now by your imagination.  So check out the [Bsquask SDK](http://github.com/nezticle/RaspberryPi-BuildRoot) and if you run into any problems, just drop me an email or file a Github issue.
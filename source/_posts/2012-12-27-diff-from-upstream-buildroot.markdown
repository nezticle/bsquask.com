---
layout: post
title: "Diference from upstream BuildRoot"
date: 2012-12-27 17:57
comments: true
categories: 
---

Today I am working on updating the Bsquask SDK to BuildRoot 2012.11 from the previous 2012.08 release.  In the process I thought it would be nice to review all of the changes that have been made to the Bsquask SDK's master branch.

Here is the whole 4000 line [diff](https://gist.github.com/4389938) in case you are interested in every detail.  The majority of the diff consists of the Linux kernel configuration, and the BusyBox configuration.

## Breakdown of the diff:

A README file that documents how to use the Bsquask SDK

A custom busybox-1.20.2 configuration, which disables the busybox init integration from being built (since we default to using and building sysinitv)

A custom linux-3.2 configuration, which is a copy of the RaspberryPi defconfig with alsa and xpad modules built into the kernel.

A service file for ntpdate, which is needed because the stock Raspberry Pi does not come with a real time clock, and thus we need to set the current time from an external source

An inittab file that specifies what to mount, what services to start, and how many tty consoles to start.

An interfaces file defining the default behavior for the Ethernet port on the Raspberry Pi is to try and acquire an IP address using DHCP.

A script that should be run after all packages are generated that sets the root password, sets the default shell to bash, packages the boot partition, and installs the custom ntpdate service, inittab, and interface files mentioned above.

A BuildRoot defconfig for building for the Raspberry Pi.  This sets up a minimal environment to cross compile and run Qt 5 applications without X11.  This includes the custom packages necessary for the Raspberry Pi (bootloader, VideoCore, custom Linux kernel)

### Bootloader   
The bootloader files for the Raspberry Pi are distributed in binary blobs provided in the Raspberry Pi [firmware repository](https://github.com/raspberrypi/firmware).  This repository can take a significant time to clone, so I've opted for packaging the necessary files for use with the script, in the interest of speeding up build times for the Bsquask SDK.

### Kernel
The RaspberryPi does have source code available to build the Linux kernel, so we do.  There are patches that have not been accepted into the upstream Linux kernel, so the kernel sources are provided [here](https://github.com/raspberrypi/linux).  Cloning this git repository locally can take nearly the same amount of time to build every other package in the Bsquask SDK, so I also package snapshots of the Raspberry Pi Linux kernel source code into a tar.gz and host it.  

### VideoCore
Right now the VidoeCore files are packaged using the pre-built hard-float version provided in the [firmware repository](https://github.com/raspberrypi/firmware).  It is possible to build these files locally now, and I have an unreleased BuildRoot recipe I'm still testing to replace the current one.  Right now these files are hosted by me as well for the same reason given above.

### OMXPlayer
OMXPlayer is build because it's nice for demoing the multimedia capabilities of the Raspberry Pi.  With this player you can play 1080p h.264 videos on the Raspberry Pi without breaking a sweat.

### Wayland
I package Wayland because I need it for QtWayland, and I intend to use this as part of lightweight window compositor in place of an X11 server.

### Qt5 Packages
The Bsquask SDK was developed so that I could run Qt5 applications on the Raspberry Pi with the minimal base required to enable all features.  Most Qt 5 modules should be packaged now, but only the basic ones are enabled by default (as these work the best on the Raspberry Pi).

### Toolchain
Probably the most controversial and troublesome thing, I define and provide a custom toolchain for building for the Raspberry Pi.  The problem was that the intended target is ARMv6 hard-float, and most pre-build toolchains are configured for either building ARMv5tel soft-float, or ARMv7 hard-float.  That leaves us stuck in the middle with a toolchains that won't work right for us.  The toolchain I provide was generated with [crosstool-NG](http://crosstool-ng.org/).  
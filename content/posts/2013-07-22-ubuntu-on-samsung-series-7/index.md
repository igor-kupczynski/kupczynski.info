---
title: Ubuntu on Samsung Series 7 Ultrabook 730U
tags:
- linux
aliases:
- /2013/07/22/ubuntu-on-samsung-series-7.html
---
Is it possible to install Xubuntu 12.04 on Samsung NP730U3E?

I've got a new laptop - Samsung 7 Series NP 730U 3E. It's 13.3'' ultrabook
with a quite decent hardware under the hood:

- Intel Core i7-3537U,
- 10 GB of RAM,
- 13,3'' Full HD Display (1920x1080),
- 128 GB SSD drive,
- Two graphic cards: AMD Radeon HD 8570M + Intel HD Graphics 4000.

This machine comes with a preinstalled Windows 8. It is a bit of an issue for
me - I heavily rely on linux both for work and for my other projects.

Without the least hesitation I decided to give Xubuntu 12.04 LTS a try. In
this post you will see what challenges I faced, what does not work and if it
can be fixed.


Challenges
----------

### Samsung UEFI Bug

It was reported that Linux can destroy some of the new Samsung ultrabooks. It
turned out, that not Linux is to blame here, but [Samsung's UEFI bios][uefi-
issues]. Just to be on a safe side and in order not to destroy my shining new
laptop I followed steeps outlined in this [stack overflow answer][so-uefi].

#### Step 1 - Backup

Firstly, I booted Windows 8 and run _SW Updater_ to get the latest version of
the firmware. Then, just in case I'll ever decide to go back to windows I made
a copy of the recovery partition (this is possible through Samsung recovery
software).

#### Step 2 - Switch to Legacy BIOS mode

This step involved pressing F4 to get to this firmware setup utility. I needed
to turn off _Secure Boot_ and _Quick Boot_ and then I was presented with an
option to switch UEFI to CSM mode. CSM stands for _Compatibility Support Mode_
and is exactly what I needed. Assured that I'm not using the UEFI and hence
there is not risk to the laptop, I carried on the installation.

[uefi-issues]: http://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface#Firmware_issues
[so-uefi]: http://askubuntu.com/questions/290670/ubuntu-on-samsung-series-7-chronos-advice-please/291011#291011
[lvm-12-10]: http://www.linuxbsdos.com/2012/09/04/full-disk-encryption-and-lvm-configuration-in-ubuntus-graphical-installer/

#### Step 3 - Installation

I used the _Alternative Installer_ for Xubuntu 12.04. I encrypt all my Linux
boxes and Ubuntu offers you an easy way to set up encrypted LVM during the
installation. With 12.04 you need an alternative installer to use this
feature. Starting from Ubuntu 12.10 they [have this option on a regular
installer as well][lvm-12-10].

I needed to fine-tune the partition sizes - Ubuntu by default creates a swap
space exceeding the size of the RAM. This makes the hibernation work - the
memory dump is stored on the swap space - but seems a bit of a waste. To
sacrifice 10GM of your precious SSD as a swap space? Especially, that I do not
use hibernation that often. Quite frankly, I've forgotten when had I last used
it. I decided to have no swap space at all. 10 GB of RAM is plenty.

OK, so with my new partition layout and BIOS mode I avoided the UEFI bug. 

### Xubuntu 12.04 Alternative Installation Issues

The alternative installer installs the packages in two steps - first the
so-called base packages and then the desktop environment, etc.

Unfortunately, it gave an error on the latter. Net effect is, I was not able
to complete the installation. From a user perspective it's a huge flaw - what
you can expect if even the installer fails. Nevertheless, after restarting the
computer it boot in text mode Ubuntu :-) I installed the Xubuntu-desktop
package:

	$ sudo apt-get install Xubuntu-desktop

And hoped that my torments are over. Not yet though - somehow it didn't want
to boot in a graphical mode...

### Xorg Not Starting

Ubuntu didn't want to boot in a graphical mode. Booting it in the text mode
revealed that there are no devices to run xorg on. So much for having two
graphical cards is a small laptop. I ended up with adding `xorg-edgers` repo:

	$ sudo add-apt-repository ppa:xorg-edgers/ppa
	$ sudo apt-get update
	$ sudo apt-get upgrade

And installing `xserver-xorg-video-intel` package:
	
	$ sudo apt-get install server-xorg-video-intel

The I created the xorg conf file:

	$ sudo vim /etc/X11/xorg.conf

With a following content:

	Section "Device"
  		Identifier "Card0"
  		Driver "intel"
  		Option "AccelMethod" "sna"
	EndSection

This did the trick and after a restart I finally saw the xfce gui.

By the way, I'm not sure if it is possible to make Linux work with both
graphic cards. It hope it is, but for now it's low on my priority list - the
Intel chipset is more than enough for my day-to-day work.

### Network Manager Not Managing WiFi

During the installation I had connected to a wifi - in order to download the
latest package updates. Don't do it if you're on the alternative installer.
The problem is that it adds the wifi to /etc/network/interfaces instead of
letting the `Network Manager` daemon manage it. What does it mean? Well, if
you're connected to the same wifi which worked during the installation then
not much. You can use the internet connection, but the network widget shows no
connections - it basically says _device not managed_.

In order to fix, you can follow [this advice][so-network-man].

Basically, you need to remove all the devices except for `loopback` from
`/etc/network/interfaces`. The file should look like this:

	auto lo
	iface lo inet loopback

And then restart your computer.
	

[so-network-man]: http://askubuntu.com/a/71205

### Touchpad with only basic functionality

Even if you are not a big fan of touchpads, sooner or later you will need to
use one. Unfortunately, Ubuntu does not fully support the touchpad built into
Samsung Series 7. There is a [bug at lauchpad][lp-1166442] which aims at
fixing it.

As on writing this article a fix was provided but not yet included in Ubuntu.
Joseph Salisbury uploaded kernels for various Ubuntu versions which include
the fix already. Just follow this [comment][lp-1166442-100]. I hope the fix
will soon make it to the main repo and this step won't be needed any more.

After installing Joseph Salisbury's kernel the touchpad works like a charm,
i.e. it supports two finger scrolling and three finger clicking.

[lp-1166442]: https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1166442
[lp-1166442-100]: https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1166442/comments/100


### Dual Monitor Support

Xubuntu 12.04 contains XFCE 4.08. It does not play too well with two monitors
- the only thing you can set is mirroring the displays. To fix it install XFCE
4.12.

Just add required repositories and do a dist-upgrade:

	$ sudo add-apt-repository ppa:xubuntu-dev/xfce-4.10
	$ sudo add-apt-repository ppa:xubuntu-dev/xfce-4.12
	$ sudo apt-get update
	$ sudo apt-get upgrade

Two displays can be now configure from XFCE System Settings and extended
desktop works without any issues.

## Conclusion

I'm quite happy with the experience. The laptop, especially with 10 GB of RAM
is a monster and a great dev box. After dealing with the problems described
above the experience is very good. I think that Samsung 7 Series NP 730U 3E
provides a really good value for the money. The performance is good, the
display is great and the case seems to be well made. The design is clear
inspired by MacBooks Pro, which may or may not be a plus for you.

You can see my desktop below.

For last two or three years I worked on 12.1'' Lenovo - with great old-school
keyboard and joystick in the middle of it. Quite frankly, I miss the
experience a bit - IMHO keyboards in Lenovo laptops are best-of-breed. But
more important - the only thing I do not like in new Samsung is the touchpad -
it's really hard to right click using it, especially when the computer in on
your lap and not on a solid surface. I think Samsung needs to work more there
to provide a better experience.


![Xubuntu 12.04](/archive/2013-07-22-desktop.png)
_Xubuntu 12.04 on Samsung 7 Series NP 730U 3E_

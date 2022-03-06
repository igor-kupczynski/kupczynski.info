---
title: Nvidia on Ubuntu 18.10 requires lightdm
tags:
- linux
aliases:
- /2018/11/18/ubuntu-1810-nvidia-lightdm.html
---
After a upgrade from 18.04 my external monitor stopped working. Installing lightdm was the solution.


**Update** 2020-11-10. [Abish Vijayan](https://disqus.com/by/abishvj/) posted a solution for Ubuntu 20.10 in the comments below.

> Made it to work. Disabled Wayland in gdm3.
> https://askubuntu.com/questions/975094/how-to-disable-wayland-in-17-10-in-gdm3-login-screen


**Update** 2019-11-15 on Ubuntu 19.10 I still need the lightdm to support multiple monitors.


## Problem

I've recently updated Ubuntu from 18.04 to 18.10 on work-issued Thinkpad p51.
The laptop has **nvidia quatro m2200** graphics it also has a 4k display and I use
an external 4k monitor most of the time.

The setup worked nicely on 18.04, but **after the upgrade to 18.10 the external
monitor was no longer detected**. The external monitor wasn't showing neither in
the *Display* settings nor in `xrandr`. Nvidia drivers are on version 390.77.

Xrandr output was something like this:

```sh
$ xrandr

Screen 0: minimum 320 x 200, current 3840 x 2160, maximum 8192 x 8192
eDP-1 connected primary 3840x2160+0+0 (normal left inverted right x axis y axis) 345mm x 194mm
## (...) all of the available resolution goes here 
```

No mention of the other monitor, not even as disconnect.

## Solution

I've googled for the solution with a little luck, until I've found [this ask
ubuntu answer](https://askubuntu.com/a/1049669/472560) suggesting to install
`lightdm` as a login manager instead of `gdm3`. It thought that it doesn't make
a lot of sense --- why the DM should influence external monitor detection? There
wasn't that many suggestions out there. so I gave it try. And it worked.

Steps:

1. Install lightdm:

   ```sh 
   $ sudo apt install lightdm
   ```

2. When asked about the display manager to use, select `lightdm`.

3. Restart and profit.

Contrary to the ask ubuntu answer, I need to stay with `lightdm`, reverting back
to `gdm3` breaks the setup again.

### But why does it work?

I've found [this
post](https://devtalk.nvidia.com/default/topic/1042491/linux/optimus-and-ubuntu-18-10-new-packages-are-good-but-lightdm-is-required-gdm3-still-broken/)
on the nvidia forum:

> The Ubuntu developer who looks after the ubuntu nvidia-prime package has
> updated his work for Ubuntu 18.10 (...) Swapping between hybrid and intel-only
> (either way) works without rebooting, you just need to log out & in again
> (...) However, it does not work well with gdm3: external monitors don't work
> because the nvidia driver crashes (...)

Now it makes sense, doesn't it?

The nvidia driver failed to load and Ubuntu switched to the integrated intel
card. But the card is not connected to any external port in this laptop, it is
nvidia that has to drive the external display.

The update to 18.10 must've updated the nvidia driver (I didn't capture the
previous version), and the new one is broken with `gdm3`.

Also, the `xrandr` output for nvidia and *disconnected* monitors:

```sh
Screen 0: minimum 8 x 8, current 3840 x 2160, maximum 16384 x 16384
DP-0 disconnected (normal left inverted right x axis y axis)
DP-1 disconnected (normal left inverted right x axis y axis)
DP-2 disconnected (normal left inverted right x axis y axis)
DP-3 disconnected (normal left inverted right x axis y axis)
DP-4 disconnected (normal left inverted right x axis y axis)
DP-5 disconnected (normal left inverted right x axis y axis)
eDP-1-1 connected primary 3840x2160+0+0 (normal left inverted right x axis y axis) 345mm x 194mm
```

It clearly lists `DP-#` which we can connect if we want to (although I'm not
sure how would I fit 6 monitors to 1 display port and 1 hdmi, but hey, the GPU
supports them).

I would have probably found the solution sooner have I noticed that `xrandr`
output is missing the `DP-# discoonected` lines, indicating we are on intel
instead of nvidia.

On the bright side, now I can switch between intel and nvidia without restarting
the laptop. Log off and on is a slight inconvenience, but also a big improvement
over the previous solution.

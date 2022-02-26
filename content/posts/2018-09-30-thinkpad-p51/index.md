---
title: Ubuntu 18.04 on Thinkpad P51
tags:
- linux
aliases:
- /2018/09/30/thinkpad-p51.html
---
Back to Linux after almost 3 year hiatus.

Switching back from MacBook to Thinkapad and Ubuntu as the main work machine.

My MBP 13'' aged quite nicely, but compiling large scala codebases and running
VMs / containers became quite frustrating. I was eligible for a refresher at
Elastic and I've decided to try Thinkpad P51 + Ubuntu. This is a 15'' beast with
64 GB of RAM. It is quite heavy, but working remotely I don't need to carry it to
the office every day, while still being able to travel if needed. I consider the
trade off acceptable.

This post is a short note and what works and a note on setting it up if I need
to do it again.

- [Out of the box experience](#sec-1)
  - [What doesn’t work?](#sec-1-1)
  - [Battery life](#sec-1-2)
  - [KDE vs Gnome](#sec-1-3)
- [Setup](#sec-2)
  - [Windows](#sec-2-1)
  - [Kubuntu Installation](#sec-2-2)
  - [After installation](#sec-2-3)
    - [Chrome](#sec-2-3-1)
    - [Nvidia](#sec-2-3-2)
    - [Keybase](#sec-2-3-3)
    - [Github ssh key](#sec-2-3-4)
    - [Emacs](#sec-2-3-5)
    - [Dotfiles](#sec-2-3-6)
    - [Zoom](#sec-2-3-7)
    - [IntelliJ](#sec-2-3-8)
    - [Spotify](#sec-2-3-9)
- [Conclusion](#sec-3)


# Out of the box experience<a id="sec-1"></a>

The out of the box experience is surprisingly good. P51 is about 1 year old, so
18.04 kernel (4.15) contains everything it needs. Basically, after the
installation I've needed only to install nvidia drivers and change display
scaling (4k screen). Second display (also 4k) works, sleep works, etc.

## What doesn’t work?<a id="sec-1-1"></a>

-   Different scaling on different displays. The laptop is 4k and the external
    monitor I use it with is also 4k. But one of them is 15'' and the other
    27'', so they have different DPI. Xorg doesn't support such a config, but
    there is a possible workaroudn with xrandr (keyword: `hidpi arch wiki`).

-   I have some apple hardware, which I quite like and wouldn't mind using with
    ubuntu. Esp. the airpods and the magic mouse. Airpods connect and you can
    use them as headphones, but the mic is off. Ubuntu doesn't consider them a
    sound input, only output. You can switch the profile from hi-fidelity
    playback to headset, but there seems to be a bug in pulse audio and
    switching the profile doesn't help (I can't find the link right now).
    Workaround is to use a wired headset, but boy that sucks.
    
-   Magic mouse 2 connects, and gives you the impression that it works, but
    there are traps. First, it disconnects every now and then; sadly the
    linux&#x2013;bluetooth experience is far from being "polished". Also, the
    scroll doesn't work (but a fix is being worked on, see
    <https://github.com/rohitpid/Linux-Magic-Trackpad-2-Driver>).

-   Firefox has laggy scrolling. The scrolling is basically broken, scroll a couple of
    lines and then it becomes *laggy* or *jerkey*. I'm not sure what to
    attribute it to, but it is super frustrating. As a workaround I've switched
    to chrome. Collegue sent me a link to arch wiki, which suggested turning off
    `smooth scrolling`. This helped I think, but I've stayed with chrome for the
    time being. The default firefox experience makes me really sad.

-   Fingerprint sensor, but hey, not like this is unexpected.

## Battery life<a id="sec-1-2"></a>

I get around 3h of regular work (compilations & build, IDE, spotify, external
display, mails, slack, I have it all). It is a far cry from MBP, but acceptable.
I plan to experiment with TLP to see if I'll get more out of it. Also, I've
haven't got a chance to test it without IDE + sbt combo; hope it'll be a bit
better in reading / surfing mode. I'm using the discrete nvidia card.

## KDE vs Gnome<a id="sec-1-3"></a>

I've played a bit with kubuntu and ubuntu live cds and decided to go with KDE.
But after a few days I've installed `ubuntu-desktop` and stick with it. I like
KDE more, but the number of configuration options and perceived stability made
me switch to gnome for now.

# Setup<a id="sec-2"></a>

## Windows<a id="sec-2-1"></a>

-   Update BIOS, firmware, etc.
-   Go to BIOS and setup hard disk password (this is really neat, hard disk is
    hardware encrypted, and you don't need to setup encrypted LVM), see
    <https://support.lenovo.com/pl/en/solutions/migr-69621>.
-   Created recovery drive for windows
    <https://support.lenovo.com/pl/en/solutions/ht117511> (in case you want to
    get back :)).
-   Download ubuntu ISO and *rusus* to “burn” the ISO on a pendrive.
-   Restart while keeping `shift` if not able to open bios.
-   In BIOS setup (well, it is called UEFI these days):
    -   Disable biometry,
    -   Use F1-F12 as primary function,
    -   Enable intel virtualization technology,
    -   Disable `quick boot`.

## Kubuntu Installation<a id="sec-2-2"></a>

-   Minimal installation (and also minimal fonts!).
-   Connected to wifi.
-   Erase the disk and install ubuntu (no need for LVM or encryption, because
    the harddisk is encrypted).
-   After the straightforward installation it went into a bootloop. I wasn’t
    really able to read the message as it was disappearing almost right away. It
    seemed to be similar to
    <https://askubuntu.com/questions/1042747/system-bootorder-not-found>
-   I’ve tried enabling and disabling the secure boot, clearing the keys, etc.
    In the end turning off quick boot to diagnostic boot did thfde trick.

## After installation<a id="sec-2-3"></a>

-   Scaling needs to be set to 2 (or you need a magnifying glass).
-   Mouse -> reverse scroll direction.

### Chrome<a id="sec-2-3-1"></a>

```sh
$ sudo apt install chromium-browser
```

### Nvidia<a id="sec-2-3-2"></a>

```sh
$ ubuntu-drivers devices

# and if you agree with the recomendation
$ sudo ubuntu-drivers autoinstall

# reboot and then you can:
## check with card is in use
prime-select query

## select one of them
sudo prime-select intel
sudo prime-select nvidia

## and reboot
```

1.  glxgears on intel

    ```sh
    $ glxgears
    367 frames in 5.0 seconds = 73.211 FPS
    301 frames in 5.0 seconds = 60.027 FPS
    301 frames in 5.0 seconds = 60.023 FPS
    301 frames in 5.0 seconds = 60.023 FPS
    ```

2.  glxgears on nvidia

    ```sh
    $ glxgears
    Running synchronized to the vertical refresh.  The framerate should be
    approximately the same as the monitor refresh rate.
    71605 frames in 5.0 seconds = 14320.992 FPS
    72945 frames in 5.0 seconds = 14588.849 FPS
    73311 frames in 5.0 seconds = 14662.161 FPS
    C73643 frames in 5.0 seconds = 14728.479 FPS
    ```

### Keybase<a id="sec-2-3-3"></a>

<https://keybase.io/docs/the_app/install_linux>

```sh
cd ~/Downloads
sudo apt install curl

curl -O https://prerelease.keybase.io/keybase_amd64.deb
curl -O https://prerelease.keybase.io/keybase_amd64.deb.sig
curl -O https://keybase.io/docs/server_security/code_signing_key.asc
gpg --import code_signing_key.asc
gpg --verify keybase_amd64.deb.sig keybase_amd64.deb
sudo dpkg -i keybase_amd64.deb
sudo apt-get install -f
run_keybase

Restart was needed to fix a permission issue while cloning a git repo
```

### Github ssh key<a id="sec-2-3-4"></a>

Follow the steps from the website

```sh
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
$ sudo apt-get install xclip
$ xclip -sel clip < ~/.ssh/id_rsa.pub
```

And [add it to the account](https://github.com/settings/keys).

Spacemacs clones some of the packages from github, and the running my config will
fail, unless this is step is done.

### Emacs<a id="sec-2-3-5"></a>

```sh
$ sudo add-apt-repository ppa:kelleyk/emacs
$ sudo apt-get update
$ sudo apt install emacs26
$ git clone https://github.com/syl20bnr/spacemacs ~/.emacs.d
$ cd ~/.emacs.d
$ git checkout develop
```

### Dotfiles<a id="sec-2-3-6"></a>

```sh
$ sudo apt install git
$ cd ~/code
$ git clone keybase://<my-dotfiles-repo>
$ cd dotfiles
$ make do-bootstrap
$ make force-install
```

### Zoom<a id="sec-2-3-7"></a>

<https://kupczynski.info/2018/09/08/zoom-and-kubuntu.html>

### IntelliJ<a id="sec-2-3-8"></a>

Install from the toolbox <https://www.jetbrains.com/toolbox/app/>

### Spotify<a id="sec-2-3-9"></a>

~As per the instructions in <https://www.spotify.com/pl/download/linux/>~

Spotify snap seems to work fine. It's in my dotfiles setup now.

```sh
$ snap install spotify

# There was a problem with scaling, spotify don’t honor the gnome scaling factor

$ sudo emacs /usr/share/applications/spotify.desktop

# Exec=spotify --force-device-scale-factor=2.0 %U
```

# Conclusion<a id="sec-3"></a>

I'm really happy with the new machine, it can handle a lot of load and the setup
was surprisingly easy. If they fix my airpods and improve the hidpi and external
monitor support I think this will be the year of linux on the laptops.

---
title: Zoom and Kubuntu 18.04
tags:
- linux
aliases:
- /2018/09/08/zoom-and-kubuntu.html
---
Fix annoying issues with Zoom and HiDPI on Kubuntu 18.04.

I've got a new laptop for my work for Elastic recently. This time I've opted for
a Thinkpad and not for a MacBook. I've installed Kubuntu 18.04. Linux on desktop
or laptop gives you a lot of opportunities to blog about something you've fixed.
Here comes a first one.

How to make [zoom](<https://zoom.us/>) &#x2014; the conferencing software
&#x2014; play nicely with kubntu?

## What's the issues?<a id="sec-1"></a>

### Super small fonts with Zoom and HiDPI display<a id="sec-1-1"></a>

Linux and HiDPI still have some way to go. In this case, the issue is that with
my KDE configuration (scaling = 2.0), zoom is so small, that it is unreadable.

The solution is simple (huge thanks to my colleagues from our internal `#linux`
room :)). Just use experiment with the `QT_SCALE_FACTOR` env variable before
starting zoom.

To make it convenient in ubuntu, I've added it to the zoom launcher:

```sh
$ sudo $EDITOR /usr/share/applications/Zoom.desktop
## Change `Exec` to something like
## Exec=/usr/bin/env QT_SCALE_FACTOR=2 /usr/bin/zoom %U
```

Before:
![Before - zoom is small](/archive/2018-09-zoom-small.png)

After:
![After - zoom is OK](/archive/2018-09-zoom-large.png)

### KDE / Kubuntu doesn't handle zoom meeting links<a id="sec-1-2"></a>

Zoom uses its own [url
scheme](https://developer.zoom.us/article/zoom-url-schemes/), e.g. open
`zoommtg://zoom.us/join?confno=123456789&pwd=xxxx&zc=0&browser=chrome&uname=Betty`
to join a meeting. Usually, even on linux, it is recognized correctly "out of
the box" (after the installation that is). In case of kubuntu, I've needed to
use a trick from the indispensable [arch
linux](https://wiki.archlinux.org/index.php/XDG_MIME_Applications) wiki:

> Tip: Although deprecated, several applications still read/write to
> `~/.local/share/applications/mimeapps.list.` To simplify maintenance, simply
> symlink it.
>
> ``` 
> $ ln -s ~/.config/mimeapps.list ~/.local/share/applications/mimeapps.list
> ``` 
>
> Note that the symlink must be in this direction because xdg-utils deletes and
> recreates `~/.config/mimeapps.list` when it writes to it, which will break any
> symbolic/hard links.

I've also issued this command first, but I think it is issued by the package
installer as well, so it wasn't needed.

```sh
$ xdg-mime default Zoom.desktop x-scheme-handler/zoommtg
```

With these two "fixes" in place, the machine is ready for the video conferencing.

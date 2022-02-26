---
title: Use wine to run windows JDK on linux
tags:
- linux
aliases:
- /2018/11/11/wine-windows-jdk-on-linux.html
---

I've got a **legacy windows app** that I need to run from time to time. The app
is written in **java**, but uses some **32 bit JNI dll**s. I've used to keep a
windows VM around to run it, but recently I was able to "port" it to
[wine](https://www.winehq.org/).

In short, you need install wine on ubuntu:

```sh
# I've needed a 32 bit version
$ sudo dpkg --add-architecture i386
$ sudo apt-get -y update && sudo apt-get install wine32
```

Then get the 32bit JDK from Oracle. It needs to be JDK, JRE wasn't working for
me (and some folks on the internet). As of time of writing I've got [JDK
8u192](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
and this is what I've tested with.

```sh
$ chmod +x ~/Downloads/jre-8u192-windows-i586.exe
$ wine ~/Downloads/jre-8u192-windows-i586.exe 
```

This starts a graphical installer. Later on, I've found out there is a
`/s` for silent mode.

Then you can run your legacy 32bit JNI windows app, on linux!

```sh
# Windows path to java:
export JAVA_EXE='C:\Program Files (x86**\Java\jdk1.8.0_192\bin\java.exe'

# JNI library path, can be relative to the working directory
export JNI=lib/win32

wine "$JAVA_EXE" -cp "legacy-app.jar" -Djna.library.path="$JNI" "com.legacy.MainClass"
```

I'm really impressed that wine can run a windows JVM. Next step is to dockerize
it, so that wine installation in not needed.

## Dokerizing "windows" 32 bit JDK

The main benefit of a docker image with 32bit windows JDK is that I don't need
to install wine and then JVM on my machine. I can run it locally or on a server
somewhere in the cloud in time. Also important, I've got a documented and
reproducible sequence of steps. And I can share them with my readers :)

### Firstly, the `Dockerfile`.

<script src="https://gist.github.com/igor-kupczynski/d6d9969bca25d98958dd2fd0f7bd208b.js?file=Dockerfile"></script>

Nothing fancy here, we've jsut scripted the steps outlined earlier. Some
improvements can be done if you care about the image size (e.g. `ubuntu` is not the
smallest base, no need to keep `jdk.exe` after the installation, etc.).

Note that I install the jvm with `/s` so this can be done headless, that is
without the X server, as there is no graphical output or buttons to click.

Before build the image, please download the JVM installer from Oracle and put it
in the same directory as the `Dockerfile`.

### To make it easier to build, lets create a `Makefile`.

<script src="https://gist.github.com/igor-kupczynski/d6d9969bca25d98958dd2fd0f7bd208b.js?file=Makefile"></script>

`build` --- builds the image with a preconfigured label.

We cloud push the image to a dockerhub, but this is a personal project for me,
so I don't want to invest in a private repo, and I'm not sure if this is
all-right with Oracle if I share their precious JVM on with my own image. For
that reason we have also `save-image` and `load-image` to easily migrate to
another machine (or stash somewhere safe to prepare for [an SSD
failure](/2018/11/02/ssd-failure.html)).


### Finally, a script to run it

<script src="https://gist.github.com/igor-kupczynski/d6d9969bca25d98958dd2fd0f7bd208b.js?file=run.sh"></script>

Again, these are just the instructions pasted above, wrapped in a container. We
need `docker run`, then `--rm` to discard the container afterwards. `-v` and
`-w` to mount the current working directory in the container. Then we just pass
the `wine` invocation.

Also note, that my legacy app reads from disk and writes to disk. It can run in
a batch mode without GUI. If you need a GUI in your app, the easiest thing to do
seems to be to mount the X socket and reuse the session. For details check [this
blogpost](http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/).

Hope this is helpful. I'm really happy that I was able to get rid of the
manual steps (and the pain of maintaining Windows VM) and turned it into
scripted, isolated and linux-friendly procedure.

---
title: Shell in Emacs - Performance
tags:
- emacs
aliases:
- /2017/12/07/mxshell-performance.html
---

Some of my coworkers use
[Eshell](https://www.gnu.org/software/emacs/manual/html_mono/eshell.html)
inside emacs as their main *terminal emulator*. I had a little bit of envy, so
I decided to give it a try. Unfortunately, I'm not going to use it as my main
shell. At least not yet.

## How fast can you print?

What is the issue then? --- performance. I've run a build which outputs a lot
of, mostly useless, warnings to the terminal. And boy, it took ages.

One can expect that outputting text in emacs maybe slower than in a dedicated
terminal emulator. But how much slower --- is it just a small annoyance or can it
eat precious minutes while waiting for chatty commands to finish?

I've found a interesting post, [Terminal and shell
performance](https://danluu.com/term-latency/), where the author measures how
much latency is added by different terminals between pressing a key and seeing
the letter on-screen. Towards the end of the article he also suggests a simple
method of measuring terminal bandwidth when it comes to the speed of printing
text:XS

> `timeout 64 sh -c 'cat /dev/urandom | base32 > junk.txt'`

> and then running

> `timeout 8 sh -c 'cat junk.txt | tee junk.term_name'`


On MacOS, you actually need install [gnu
coreutils](https://www.gnu.org/software/coreutils/coreutils.html) to get
`timeout` and `base32`. I've installed it via homebrew so I need to prefix the
utils with `g-`.

First we generate a stream of random `junk.txt`

```
$ gtimeout 1 sh -c 'cat /dev/urandom | gbase32 > junk.txt'
$ ls -lah | grep junk.txt
.rw-r--r--  1.7G igor  6 Dec 23:59  junk.txt
```

Next, we print it on various terminals for 8 seconds and at the same time
write to a file.

```
$ gtimeout 8 sh -c 'cat junk.txt | tee junk.<terminal-name>'
```

Then we can simply compare the size of the files and estimate printing
throughput based on that.

## Results

I've tested standard MacOS terminal emulator, [iTerm2](https://www.iterm2.com)
and emacs' `M-x shell` and `Eshell`.

The results are not super scientific, as I had just one run, so take them with
a grain of salt. Having said that, boy the differences are huge.

```
.rw-r--r--  245k igor  7 Dec  0:04 junk.eshell
.rw-r--r--   87M igor  7 Dec  0:02 junk.iterm
.rw-r--r--  195M igor  7 Dec  0:03 junk.macosterminal
.rw-r--r--  245k igor  7 Dec  0:03 junk.mxshell
```

| Terminal emulator | File size [MB] | Estimated throughput [MB / s] | Times faster than Eshell |
|-------------------|----------------|-------------------------------|--------------------------|
| MacOS Terminal    | 195            | 24,38                         | * 813                    |
| iTerm2            | 87             | 10,88                         | * 363                    |
| Eshell            | 0,24           | 0,03                          | * 1                      |
| M-x shell         | 0,24           | 0,03                          | * 1                      |


## Final comments

If you're into chatty builds, then go with the default terminal
emulator. Arguably, the build I mention here is misconfigured. The warnings
can (and should) be printed only if they are actionable. And the actionable
ones should be fixed. Nevertheless, since I spend so much time in the terminal
I will encounter chatty scripts in the future. The tool should be able to
handle the reality not the other way round.

Note that, there are aspects other than the throughput. According to the post
linked above, `Eshell` has quite small and consistent latency. It is quite
snappy when it needs to send `sigkill`. Plus, you get all the benefits of
emacs and elisp. This may be more important to some than the throughput.

There are optimizations that can be made to speed emacs text rendering, but I
haven't tried them yet. Anecdotal evidence suggests they won't improve the
throughput 400 times.

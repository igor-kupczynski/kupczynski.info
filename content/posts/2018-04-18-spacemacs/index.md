---
title: Learning VIM with Spacemacs
tags:
- emacs
aliases:
- /2018/04/18/spacemacs.html
---
I just don't grok the vi (yet).

![Spacemacs](/archive/2018-04-spacemacs.png)

Three weeks ago, I've ditched most of my emacs config and installed
[spacemacs](http://spacemacs.org). The main driver was that I wanted to try
modal editing instead of emacs key chords. I could've tried vim of course, but
I'm already invested in emacs and feel at home there.

# Evil-mode<a id="sec-1"></a>

Luckily, there is a brilliant vim emulation in emacs, called [evil
mode](https://github.com/emacs-evil/evil). I didn't want to spend too much time
configuring it though plus I have no idea which vim plugins ported to emacs are
worth looking at, so I've given spacemacs a go. Spacemacs is a distribution of
emacs preconfigured with evil and with an extensive documentation.

Actually, I'm positively surprised how polished it is. Note: I'm using `develop`
branch. Whether you're an emacs user who wants to try vim-style editing or a vim
user who wants to switch to emacs and its superior package ecosystem (well, I
just <3 magit) then spacecmacs is worth giving a shot.

*Note to future self: [doom-emacs](https://github.com/hlissner/doom-emacs) is
another evil-preconfigured distribution which gets a lot of good feedback
recently*.

# Zen of Vim<a id="sec-2"></a>

The only problem is that I just don't know vim :) Of course, I can change a
config option on a remote server or scroll through a config file. But my vim
knowledge is super limited, I don't know useful shortcuts or even the
philosophy.

I've found this a nice post on StackOverflow about the [Zen of
vim](https://stackoverflow.com/a/1220118/320537).
This motivated me to go through the first series of wonderful vim screencast by
Derek Wyatt. My notes form the videos are below in case I've ever need to
reference them.

# How do I like it so far?<a id="sec-3"></a>

Well, the modal editing is quite a change. I think that after three weeks I'm
still a bit less efficient. Actually, the navigation in *normal* mode feels
quite efficient, even if I don't have the muscle memory yet. I struggle a bit
more with writing (like this blog post), the mental shift from modeless,
chord-full editing is a bit harder.

On the upside, there is a good IntelliJ plugin emulating vim, so I can work on
my muscle memory more :) *I use IntelliJ to write Java / Scala at work*.

Again, the Derek Wyatt's set of videos was super helpful (I've checked the
*novice*, for now, once I'm comfortable with that I'll probably go further).

What've started as an experiment, to see if there is a better way may become may
permanent mode of operation. Stay tuned.


# Learning to vi

## Favorites

Some of my favorite vi / spacemacs tricks

- **`*  #`** --- search for word under cursor forward / backwards.
- **`qa  q  @a`** --- start recording macro under `a`, finish recording, replay. Of
  course emacs has macros as well, but the vi-style shortcuts are super easy to
  remember and I use them more often.
- **`:%s/from/to/g`** --- search and replace in the buffer. Optional `c` for
  confirmation.
- **`C-v`** --- visual block selection. Also, the commands you perform work in a
  multicursor way.
- **`.`** --- repeat last editing command.

## Vim Videos by Derek Wyatt<a id="sec-4"></a> ##

<http://derekwyatt.org/vim/tutorials/>

### [BASIC Movement (Screencast 1)](https://vimeo.com/6170479)<a id="sec-4-1"></a> ###

| Key          | Description                                                           |
|--------------|-----------------------------------------------------------------------|
| `h  j  k  l` | arrows                                                                |
| `0  $`       | start / end of line                                                   |
| `w  W`       | next word / WORD                                                      |
| `e  E`       | end of word / WORD                                                    |
| `b  B`       | start of word / WORD backwards                                        |
| `ge  gE`     | end of previous word / WORD                                           |
| `f<char>`    | follow <char> &#x2014; search for next occurrence of <char>           |
| `F<char>`    | follow backwards                                                      |
| `t<char>`    | until <char> &#x2014; similar to f<char> but places the cursor before |
| `T<char>`    | until backwards                                                       |
| `;`          | repeat f/F/t/T movement                                               |
| `gg`         | go to the top                                                         |

-   **`word`:** keyword, doesn't include `.`, `_` and other special characters
-   **`WORD`:** up to next space
-   **`count<operator>`:** the movement commands can take count before them

### [BASIC Movement (Screencast 2)](https://vimeo.com/6185584)<a id="sec-4-2"></a> ###

| Key                | Description                                                  |
|--------------------|--------------------------------------------------------------|
| `C-f  C-b`         | full page forward / backward                                 |
| `C-u  C-d`         | half page up / down                                          |
| `H  M  L`          | head / middle / last line on screen                          |
| `gg  G`            | top / bottom                                                 |
| `17G`              | go to line 17                                                |
| `*  #`             | search for word under cursor forward / backward (then n / N) |
| `/<term>  ?<term>` | search for <term> forward / backwards                        |
| `zz`               | center on screen                                             |

### [Basic Movement (Screencast 3)](http://vimeo.com/6216655)<a id="sec-4-3"></a> ###

| Key      | Description                            |
|-------- |-------------------------------------- |
| `]]  [[` | next / previous `{` in 0th column     |
| `][  []` | next / previous `}` in 0th colunn      |
| `%`      | matching parentesis                    |
| `:marks` | list current marks                     |
| `m<char>` | mark position as <char>                |
| `'<char>` | jump to line of mark <char> at ^       |
| `\<char>` | jump to line and column of mark <char> |
| `''`      | jump to previous location              |

### [Basic Editing (Screencast 1)](https://vimeo.com/6329762)<a id="sec-4-4"></a> ###

| Key       | Description                                                     |
|--------- |--------------------------------------------------------------- |
| `i  I`    | start inserting under the cursor / at the beginning of the line |
| `a  A`    | append after the cursor / the line                              |
| `o  O`    | open line below / above current                                 |
| `x  X`    | delete / backspace                                              |
| `d<motion>` | delete <motion> (e.g. `dw` delete word)                         |
| `dd`      | delete line                                                     |
| `.`       | repeat last editing command                                     |
| `c<motion>` | change <motion> (e.g. `cw` change word)                         |
| `C`       | change to the end of the line                                   |
| `R`       | replace (overwrite) mode                                        |
| `r`       | replace character under cursor                                  |
| `s`       | substitute (replace a character and put you into insert mode)   |
| `S`       | change whole line                                               |

### [Basic Editing (Screencast 2)](http://vimeo.com/6332848)<a id="sec-4-5"></a> ###

| Key       | Description                                              |
|--------- |-------------------------------------------------------- |
| `yy  Y`   | yank line                                                |
| `y<motion>` | yank <motion>                                            |
| `p`       | paste after (cursor / line)                              |
| `P`       | paste before (cursor / line)                             |
| `J  gJ`   | join line / without space                                |
| `v  V`    | visual selection by character / line                     |
| `C-v`     | visual block (then `I` to insert at the beginning, etc.) |
| `gv`      | re-do last visual selection                              |

### Working with many files<a id="sec-4-6"></a> ###

*A bit less useful from spacemacs perspective, but surprisingly most of the keys
still work; I've aggregated the screencasts here*

[Vimeo #1](https://vimeo.com/6306508)

[Vimeo #2](https://vimeo.com/6307101) (not applicable to spacemacs)

[Vimeo #3](https://vimeo.com/6342264) (about windows, surprisingly applicable)

| Key                     | Description                                                                         |
|----------------------- |----------------------------------------------------------------------------------- |
| `:ls`                   | list buffers (helm in spacemacs)                                                    |
| `:b <name>`             | switch to <name> (w/o name to alt buffer, in vim this is `:b#`)                     |
| `:bd <name>`            | delete <name> (current buffer without name)                                         |
| `C-w`                   | leader key for windows, usually equivalent to `SPC w` in spacemacs, noted otherwise |
| `C-w o  SPC w m`         | only buffer (maximize buffer in spacemacs)                                          |
| `:sp  :split  (<fname>)`  | split horizontally                                                                  |
| `C-w s  SPC w -`        | .same                                                                               |
| `:vsp  :vsplit (<fname>)` | split vertically                                                                    |
| `C-w v  SPC w /`        | .same                                                                               |
| `SPC w M`               | exchange windows (the closes equivalent to `C-w x` in vim)                          |
| `C-w hjkl`              | switch to window right / down / up / left (accepts count)                           |
| `C-w HJKL`              | move window right / down / up / left                                                |
| `C-w c  SPC w d`        | close window (delete window in spacemacs)                                           |
| `C-w p  SPC w w`        | previous window (other window in spacemacs)                                         |



## Why, oh WHY, do those #?@! nutheads use vi?

Another zen-of-vim style [article](http://www.viemu.com/a-why-vi-vim.html).

* Always exit back to normal mode.

* Use **`.`** to repeat the last editing command.

* **`%`** matches the other parenthesis, sure. But what if you're not on a one?
  It will go forward until it finds a parenthesis and then goes it is matching
  pair. It is nice combined with editing, like **`c`** or **`d`**.
  
* **`i`**, **`a`** --- motions, but can be only used after **`c`**, **`d`** or
  **`v`**. **`i>`** is *inner angle bracket block*, e.g. **`di>`** delete inner
  angle bracket block. **`a`** is similar, but also deletes the parenthesis.
  This can be used with **`'`**, **`"`**, **`{`**, **`(`**, **`[`**, **`<`** or
  **`B`** (for block), **`t`** (for tag), **`w`** (for word).

  * Similarly, **`s`** --- only after **`c d v`**. E.g **`dst`** --- delete
  surrounding tag, **`ds>`** --- delete surrounding angle brackets, etc.
  
  * So, **`i`** *inner*  +  **`s`** *surrounding*  =  **`a`**.

* **`zz  zt  zb`** --- scroll current line to the center / top / bottom.

* **`>`** --- intend region selected by next motion, e.g. **`>aB`** intend
  current block.

* **`]p`** --- paste and auto intend.

# Spacemacs

| Key         | Description                |
|-------------|----------------------------|
| `SPC f e d` | opens ~/.spacemacs         |
| `SPC t L`   | enables line wrapping      |
| `SPC f f`   | opens file                 |
| `SPC t n`   | shows line number          |
| `SPC b`     | shows action for buffer    |
| `SPC b d`   | kills current buffer       |
| `SPC TAB`   | cycle between open buffers |
| `SPC SPC`   | M-x                        |

## Org mode

| Key         | Description                          |
|-------------|--------------------------------------|
| `SPC a o o` | shows org mode agenda                |
| `, d`       | set deadline for the current TODO    |
| `, s`       | set schedule for the current TODO    |
| `SPC m RET` | insert headline (inherit from above) |
| `C-c C-t`   | toggle status for TODO               |
| `C-c [`     | add this buffer to agenda list       |
| `C-c ]`     | remove this file from agenda list    |
| `, A`       | archive a TODO item                  |
| `, R`       | refile an archived item              |


# Doom emacs

| Key         | Description                         |
|-------------|-------------------------------------|
| `SPC p p`   | open project                        |
| `SPC SPC`   | open other file in the same project |
| `SPC ,`     | switch to workspace buffer          |
| `SPC w v`   | split windows side-by-side          |
| `C h/j/k/l` | move between windows                |
| `SPC w c`   | close the window                    |
| `SPC o n`   | neotree                             |
| `SPC o t`   | terminal                            |

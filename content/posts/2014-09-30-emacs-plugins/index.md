---
title: Two Recently Discovered Emacs Plugins
tags: []
aliases:
- /2014/09/30/emacs-plugins.html
---
Edit your gmail messages with emacs and make http requests from your favorite editor

I recently found two interesting emacs plusing which can be applied
for many usecases.

Gmail Message Mode
==================

[`gmail-message-mode`][gmm] lets you invoke an emacs editor straight
from the compose screen in gmail. You need to click on an icon or you
can use a keyboard shortcut - `alt + enter` (this key chord is not
customizable, its just on/off, what a misery for emacs geeks ;-)).

![1. New email](/archive/2014-09-new-email.png)

Then it'll open a new emacs frame. You can use markdown syntax.

![2. Write in emacs](/archive/2014-09-emacs.png)

When you're done just hit `C-x #` and the buffer is converted to html
and inserted in the gmail compose window.

![3. Generated email](/archive/2014-09-gen-email.png)

You need to have a markdown converter installed, which will respond to
`markdown` command. In my case this is [marku].

    $ markdown --version
    Maruku 0.6.1

There are two pieces to this plugin - one is the
[browser extension][plugin] and the other is a piece of
[emacs code][emacs-server]. Both are easy to install.

This plugin works fairly well, but because of interplay between
markdown, html and gmail there are some glitches sometimes. Maybe I
need to try it with a different markdown converter.


[gmm]: https://github.com/Bruce-Connor/gmail-mode/
[plugin]: https://chrome.google.com/webstore/detail/edit-with-emacs/ljobjlafonikaiipfkggjbhkghgicgoh
[emacs-server]: http://melpa.milkbox.net/#/edit-server


Restclient
==========

[`Restclient`][rc] allows you to write and annotate http requests in a
text file and then run them at will. Perfect for documentations
purposes of for experimenting with a REST API.

The usage is very simple. Write a bunch of requests, separate them by
comments (lines starting with `#`). Select a request and hit
`C-c`. You can use http methods and customer headers.

![Emacs Rest Client](/archive/2014-09-restclient.png)

To use basic auth, you can use `Authorization` header like this:

    Authorization: Basic user:password

Then select `user:password` and hit `M-x base64-encode-region`

    Authorization: Basic dXNlcjpwYXNzd29yZA==

[rc]: https://github.com/pashky/restclient.el

This is perfect to document an API with examples, or just to have a
record of a session with an API.

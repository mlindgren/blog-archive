---
layout: post
title: "Configuring DPI in Lubuntu/LXDE"
date: 2015-02-21 20:36:28 -0800
comments: true
categories: ["Technology"]
---

If you've tried to use [Lubuntu](http://lubuntu.net/) (or
[LXDE](http://www.lxde.org/) on another distribution) with a high-DPI display,
you've probably noticed that fonts and other UI elements are so tiny as to be
illegible without a magnifying glass.  You've probably also noticed that there
is no GUI with which to adjust the UI scaling factor. Happily, it is possible to
change the DPI settings in LXDE, but this being Linux, it requires editing
obscure configuration files. Here's what you'll need to do:

1. In your home directory, create a new text file named `.Xresources`
1. In this file, enter your desired DPI in the following format: `Xft.dpi: 150`
1. Restart the X server. You can do this by pressing `Ctrl + Alt + F1` to enter
single-user mode, then running `sudo service lightdm stop`, and then `sudo
service lightdm start`. (Note that it may be a different service if you are not
running Lubuntu. Alternatively, you can just reboot your machine.)

This will scale UI elements in _most_, but not all, applications. For instance,
it doesn't resize the desktop panel, so you'll likely want to do that as well.
Luckily you can do so easily by right clicking on an empty space on the panel
and selecting Panel Settings; from there, just change the height of the panel in
pixels to a suitable value.

This is yet another tip that I'm posting mainly becuase it took me an inordinate
amount of time to figure out how to do it. I found many suggestions on how to
enable scaling, but none of them worked until I stumbled across the above
instructions on the blog of [bebabi34](http://bebabi34.altervista.org/index.php?action=viewnews&news=1380636261).
His blog is in Italian, so naturally it's not very searchable for English
speakers; hopefully by reproducing his instructions here, I can save others some
time.

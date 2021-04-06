---
layout: post
title:  "Tip: improve your mouse accuracy"
date:   2021-04-06 15:00:00 UTC
---

Each operating system has its own rules for determining how far mouse pointer needs to go when you move the mouse. The default settings most often use mouse acceleration – an idea that the pointer should respond not only to the distance traveled by the mouse, but also the speed. The faster you move the mouse, the farther the pointer goes.

# The problem with mouse acceleration

Mouse acceleration makes it harder to develop muscle memory to hit far away targets accurately. You need to be precise not only in the distance you move the mouse but also the speed, which is harder to do consistently. The other issue is that acceleration curves are different on each operating system, so you'll have to get used to each one.

# Solution: flat pointer acceleration curve (aka no acceleration)

Flat acceleration curve ignores mouse velocity completely and only applies some constant factor to the distance and direction the mouse moves. What is more, it's completely the same on all operating systems, which is a great thing for people who regularly use different operating systems.

# Setting it up

macOS

```sh
defaults write -g com.apple.mouse.scaling -integer -1
```

Linux (GNOME)

```sh
gsettings set org.gnome.desktop.peripherals.mouse accel-profile flat
```

Windows

Open the "Mouse Properties" control panel dialog, uncheck "Enhance pointer precision" in the "Pointer Options" tab.

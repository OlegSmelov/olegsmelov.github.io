---
layout: post
title:  "Electron includes an Xbox controller driver"
date:   2026-07-28 00:00:00 UTC
---

Every so often I remember the great [Electron is flash for the desktop](https://josephg.com/blog/electron-is-flash-for-the-desktop/) blog post which mentions that on macOS, Chrome (and by extension Electron) includes a user-space Xbox controller driver:

> And its not a stretch to call chrome an OS. By lines of code, chrome is about the same size as the linux kernel. [...] On MacOS it even contains a userland USB driver for xbox360 controllers. (I know its there because I wrote it. Sorry.)
>
> Does slack contain my code to use xbox controllers? Does the slack team know? Does anyone know?

Until one day I realized that the question if Electron contains an Xbox driver is 100% answerable. Let's take Obsidian as an example:

```
$ cd "/Applications/Obsidian.app/Contents/Frameworks/Electron Framework.framework/Versions/Current"
$ strings "Electron Framework" | grep "Error listening for Xbox controller add events:"
Error listening for Xbox controller add events:
Error listening for Xbox controller add events:
```

This message appears in the source code for Chromium in [xbox_data_fetcher_mac.cc](https://github.com/chromium/chromium/blob/8048cf8c5d1a55937d162ae36728b260e0870343/device/gamepad/xbox_data_fetcher_mac.cc#L239). The same binary also contains the name of the source file:

```
$ strings "Electron Framework" | grep xbox_data_fetcher_mac.cc
../../device/gamepad/xbox_data_fetcher_mac.cc
../../device/gamepad/xbox_data_fetcher_mac.cc
```

Which means Electron *does* contain an Xbox controller driver. Case closed. Figuring out how to make use of it is a rabbit hole for another day.

P.S. Someone please make an extension for VS Code to add controller support. 😛

---
layout: post
title:  "Notes on building computers"
date:   2019-11-22 00:00:00 UTC
---

I've had to build a few computers for work, mainly to be used as Linux workstations. It didn't always go smoothly, so let's go through some things I've learned.

### New hardware is unproven hardware

If you buy a new piece of hardware, you're more likely to encounter issues with both hardware and software. We've bought a couple of RTX 2080 Ti GPUs, and we've had problems with failing hardware and misbehaving drivers.

### Memory

When choosing RAM, look for RAM confirmed to work with your motherboard. Motherboard manufacturers usually publish Memory and CPU compatibility lists called QVL on their websites. Buy the exact model specified.

### Extreme Memory Profile (XMP)

Even though DDR4 RAM is branded as having speeds such as 3200 MHz, usually it will only run at 2133 MHz out of the box. You'll have to enable XMP in your BIOS to unlock the full speed.

XMP is marketed as stable manufacturer-approved settings for your RAM. However, I've had multiple cases where XMP caused system instability. **Treat XMP as overclock.** If you're experiencing random reboots, crashes, computer not waking up from sleep, monitor not turning on, this could all be caused by XMP. Run at stock speed for a while and see if the problem goes away.

If XMP didn't work, you can try overclocking RAM yourself. The process is quite complicated, though, as there are a lot of other settings besides clock speed.

### RAM slots

I've recently learned that it might also matter which of the two slots of the same channel you use. For the last computer I've built, I've had to use A2 and B2 slots instead of A1 and B1 to make RAM work at full speed.
 
Refer to motherboard's manual for what manufacturer recommends. Here's an example from the manual for ASUS TUF GAMING X570-PLUS:

![Recommended memory configurations for X570 motherboard](/assets/x570-memory-configurations.png)

### Run a benchmark

It's not enough to order the parts and put them together. You also want to make sure your computer performs as expected, and that's what benchmarks are for.

I recommend [Geekbench][2]. It's free if you don't mind publishing your benchmark results on the internet. If your scores are significantly lower than the typical result for your CPU, you have a problem somewhere.

### Run a stress test

Benchmarks can show that your computer is fast, but is it stable? I usually run [Prime95][3] in stress test mode for up to a day.

### How to debug stability issues

After a crash caused by instability, you are unlikely to find any relevant logs after a reboot. However, logs are extremely useful when trying to figure out what went wrong. To access kernel logs, you need to [send logs to another computer using netconsole][0].

If you're having issues with suspend, there's a great guide on [how to debug suspend issues][1].


[0]: https://wiki.archlinux.org/index.php/Netconsole
[1]: https://01.org/blogs/rzhang/2015/best-practice-debug-linux-suspend/hibernate-issues
[2]: https://www.geekbench.com/
[3]: https://www.mersenne.org/download/

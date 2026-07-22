---
layout: post
title:  "Adding hardware debounce to Das Keyboard 4"
date:   2026-07-22 00:00:00 UTC
---

Turns out [replacing the rotary encoder]({% post_url 2026-07-06-das-keyboard-volume-knob %}) did not fix the root cause of the problem, and the volume knob was still registering multiple increments. After struggling to understand why, I decided to disassemble another Das Keyboard 4 that I own from around 2019 that has never had this problem despite the heavy use.

After looking at both the old and the new PCBs I've spotted something: on the new board pull-up and series resistors seem to have been misplaced.

![Old and new revision compared]({{ "/assets/dk-revision-comparison.jpg" | absolute_url }})

Left side is the old PCB, right side is the new broken one. The slight differences in values are not important here (1.2 kOhm vs 820 Ohm), the important thing is that the two outside resistors are the two pull-up resistors, and the two middle ones are series resistors that connect the output pins of the encoder to the microcontroller.

On the old board, the pull-up resistors are both 4.7 kOhm, and the series resistors are 1.2 kOhm. On the new board one channel doesn't look right: the pull-up resistor is 820 Ohm, and the series resistor is 4.7 kOhm.

Here's a crude schematic overlaid on top of the photo:

![PCB circuit schematic]({{ "/assets/dk-encoder-channels.jpg" | absolute_url }})

Doesn't look intentional to me, must have been a mistake :) This put some doubt in my mind that if they could mess up resistors in this revision, perhaps they also messed up debounce. The microcontroller is different on the new board, availability issues might have been the reason for the new revision.

The bottom two resistors need to be switched around.

## Adding an RC filter

What's missing for the debounce filter is a capacitor, and the existing series resistors should work great for the RC filter. Fortunately, to the right of the series resistors is a huge ground plane that we could solder to. I ordered some 0603 capacitors, ended up using 220 nF ones.

Here's an in-progress photo:

![In-progress photo of adding a single capacitor]({{ "/assets/dk-debounce-progress.jpg" | absolute_url }})

I soldered the two capacitors to the right side of the series resistors, then scraped away the solder mask right next to the new capacitors, then soldered them to the ground plane.

![Final result with two capacitors fully soldered]({{ "/assets/dk-final-result.jpg" | absolute_url }})

Very happy with the final result, looks great :) This is the first time I repaired something not just by replacing a broken part, but by fixing a design issue.

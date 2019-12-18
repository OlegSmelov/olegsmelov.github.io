---
layout: post
title:  "Stress testing suspend on Linux"
date:   2019-12-18 00:00:00 UTC
---

Here's a script you can use to stress test suspend on Linux:

```sh
#!/bin/sh

# Watch kernel logs for the word `corrupt`
dmesg -w | grep --color -i corrupt &

echo "Starting stress test in 5 seconds..."
sleep 5

# Stress test by repeatedly putting computer to sleep for 10s
for i in $(seq 100); do
  echo "Iteration #$i..."
  rtcwake -m mem -s 10
  sleep 10
done
```

Save the file as `stress-test-suspend.sh`, then run:

```sh
$ chmod +x stress-test-suspend.sh
$ sudo ./stress-test-suspend.sh
```

The idea comes from the page [Best practice to debug Linux* suspend/hibernate issues][0].


[0]: https://01.org/blogs/rzhang/2015/best-practice-debug-linux-suspend/hibernate-issues

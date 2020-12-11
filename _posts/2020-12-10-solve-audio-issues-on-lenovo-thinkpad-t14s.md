---
title: Solve audio issues on Lenovo Thinkpad T14s
date: 2020-12-10 01:00:00 +0100
categories: [Linux, audio]
tags: [linux, manjaro, audio]
---

I spent more than an hour on this issue. I just got a Lenovo T14s and my sound card was not detected with `alsa-lib` 1.2.4-3 on Manjaro Linux. I was getting a "dummy output" in PulseAudio's Volume Control (`pavucontrol`). 

After trying various things, what solved the problem was installing `sof-firmware` and rebooting. Whew! 
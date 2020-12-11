---
title: Converting .m4a and .wma to .mp3
date: 2008-02-25 00:00:00 +0100
categories: [Linux, audio]
tags: [linux, mp3, m4a, wma]
math: true
---

[This is a synthesis of two posts I wrote on [Blogspot](https://plug-and-pray.blogspot.com/) in 2008.]

I've compiled here two recipes for converting to `.mp3`.

## Convert .m4a to .mp3

Install `lame` and `faad`, then `cd` into a directory containing `.m4a` files that you wish to convert to `.mp3` and run:

```bash
for x in *.m4a ; do faad -o - "$x" | lame -V 0 - "${x%m4a}mp3" ; done
```

## Convert .wma to .mp3

Install `lame` and `mplayer`, then `cd` into a directory containing `.wma` files that you wish to convert to `.mp3` and run:

```bash
for x in *.wma ; do mplayer -vo null -vc dummy -af resample=44100 -ao pcm:waveheader "$x" ; lame -m s audiodump.wav -o "${x%wma}mp3" ; rm audiodump.wav ; done
```


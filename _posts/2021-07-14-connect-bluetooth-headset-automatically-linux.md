---
title: Connect a bluetooth headset automatically in Linux
date: 2021-07-14 00:00:00 +0100
categories: [Linux, audio]
tags: [linux, audio, bluetooth]
---

I can never remember how to automatically connect to my bluetooth headset so here is a very short recipe.

Make sure your bluetooth headphones are already paired and trusted. Then, assuming you are using PulseAudio, edit `/etc/pulse/default.pa` as explained [here][ArchLinux wiki] and add the following to it:

```
### Automatically switch to newly-connected devices
load-module module-switch-on-connect
```

Now, the next time you switch on your bluetooth headphones, they will automatically connect and become the default device.

<!-- links -->

[ArchLinux wiki]: https://wiki.archlinux.org/title/bluetooth_headset#Setting_up_auto_connection

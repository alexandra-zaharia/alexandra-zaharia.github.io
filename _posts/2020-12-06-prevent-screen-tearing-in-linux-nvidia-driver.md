---
title: Prevent screen tearing in Linux using the NVidia driver
date: 2020-12-06 01:00:00 +0100
categories: [Linux, video]
tags: [linux, manjaro, nvidia]
---

I spent the whole afternoon for this very easy to fix problem. I needed to install the proprietary NVidia driver instead of the open-source one (because Unreal Engine 4 and because DaVinci Resolve, that's why). However my desktop was utterly useless afterwards because of an obscenely low refresh rate that was literally making my head spin. The horrible screen tearing had nothing to do with the falsely reported 144 Hz refresh rate of my desktop. The open-source Nvidia driver was actually capable of providing this refresh rate, but not the proprietary NVidia one... out of the box.

Worse than all was the fact that this was happening with the latest driver (`video-nvidia-455xx`) and not many people seemed to have this issue. 

After spending an extremely frustrating afternoon, I found the solution:

![NVidia settings](/assets/img/posts/nvidia_settings.png)

You can either add the following line directly to your `nvidia.conf` file (at `/etc/X11/mhwd.d/nvidia.conf` in Manjaro Linux) under the Screen section (adjust the resolution and the refresh rate to your screen's specs):

```
Option "metamodes" "2560x1440_144 +0+0 {ForceFullCompositionPipeline=On}"
```

Or you can launch `nvidia-settings` as root (because it needs to be able to write the new settings to `nvidia.conf`) and enable the "Force Full Composition Pipeline" option. 

(Rant: I don't know what's worse, the VLC or the NVidia Settings UI...) Voila. The end. 
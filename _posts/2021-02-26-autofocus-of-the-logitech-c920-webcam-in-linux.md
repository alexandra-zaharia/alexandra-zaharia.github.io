---
title: Autofocus of the Logitech C920 webcam in Linux
date: 2021-02-26 22:43:00 +0100
categories: [Linux, webcam]
tags: [linux, webcam]
---

![Logitech C920 HD Pro](/assets/img/posts/logitech_c920.jpg){: width="400"}

In a [previous post][pp] I wrote about how I set up my webcam on Linux. It turned out it was actually suffering from random disconnects which made it very unreliable for professional video calls. I also couldn't get more than 0.5 fps if I recorded with it, so I switched to the (very expensive for what it's worth) Logitech C920 HD Pro.

## Too expensive

The Logitech C920 does have its advantages:

* It works on Linux out of the box
* It doesn't seem to randomly disconnect (so far) 
* Allows recording 1080p at 30 fps

But apart form that I actually find its video sensor inferior to that of the cheap Chinese webcam:

* The contrast is off no matter what I do 
* The colors seem artificial (although with enough tuning of the saturation and white balance temperature you can get something decent)
* The viewing angle is large, not extra large

It also lacks some features with respect to the cheap Chinese webcam:

* It doesn't swivel (if you want to rotate it, you have to rotate your screen)
* It doesn't come with a privacy shutter
* It doesn't come with a tripod

But I guess this is the "package" you get when buying into a "reputable" brand like Logitech... anyway, I digress. The real issue is...

## Bad autofocus

The autofocus on this webcam is horrible! I did myself a favor and promptly turned it off. With the `v4l-utils` package installed, do this (assuming your Logitech C920 is `/dev/video0`):

```
v4l2-ctl -d /dev/video0 --set-ctrl=focus_auto=0
v4l2-ctl -d /dev/video0 --set-ctrl=focus_absolute=0
```

Add the above lines to a script to run automatically each time you log in (put it in `/etc/profile` or add it as one of the scripts to run in `/etc/profile.d/`). If the image is not sharp enough, you can also alter the `sharpness` parameter (see below).

## Other settings

To see the other settings that are available for the Logitech C920 webcam:

```
v4l2-ctl -d /dev/video0 --list-ctrls-menus
```

You can manually tune the brightness, contrast, saturation, white balance temperature, sharpness, exposure and gain settings with `v4l2-ctl`. The pan and tilt don't seem to work. 

If you need a graphical user interface, you can use `guvcview`.

## Conclusion

All in all it's not a bad webcam. I just wish the sensor was as powerful as the one in the Chinese webcam but without its stability (random disconnect) issue.


<!-- links -->
[pp]: {% post_url 2020-12-04-getting-the-most-out-of-your-webcam-on-linux %}
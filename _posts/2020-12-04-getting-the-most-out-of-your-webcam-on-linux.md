---
title: Getting the most out of your webcam on Linux
date: 2020-12-04 19:38:00 +0100
categories: [Linux, webcam]
tags: [linux, webcam]
---

**\[UPDATE February 26 2021: This webcam didn't fare so well after all. I switched to a [Logitech C920][np] after three months due to unacceptable random disconnects.\]**

![Chinese webcam on Amazon](/assets/img/posts/webcam.jpg){: width="400"}

So... I bought a [Chinese webcam][]. And it's not that bad, now that it works. See, the first time I plugged it in, `dmesg` wouldn't stop complaining:

```
[65184.538621] usb 2-9.5: 3:1: cannot set freq 16000 to ep 0x83
```

Some frequency couldn't be assigned to _something_ in the webcam's hardware. End result: my webcam didn't provide any `/dev/video0` device. Not cool.

## Getting the webcam to work under Linux

I was seriously considering returning it to Amazon, but I first needed to check out whether it worked on Windows. Unsurprisingly, it did. Surprisingly, when Chinese vendors say their device is _plug-and-play_, what they mean is _Plug. Wait for OS to install drivers. Play (eventually)_. Lesson learned!

After making sure the webcam was functional in Windows 7, I rebooted to Linux and, lo and behold, the webcam was recognized with no problems. This means that the Windows drivers must have also initialized the webcam by actually writing a frequency to its registers. Linux was content enough with this change, since I finally obtained what I wanted in `dmesg`, that is the golden line stating that the camera was finally providing a video device:

```
[77945.854333] input: HD Web Camera: HD Web Camera as /devices/pci0000:00/0000:00:14.0/usb2/2-9/2-9.5/2-9.5:1.0/input/input33
```
## Checking the webcam's video

It was time to take a look through the webcam's lens. Instead of firing up a Google Meet call, you can quickly check out how your video looks like with a handy little tool called `cheese`. It can also be used record videos or take pictures through the camera. However, for some weird reason, when I attempt to record from the webcam with `cheese`, the video stops after a few seconds and sometimes even VLC cannot read it. See below _Recording from webcam_ for possible solutions.

## Adjusting the webcam's video

The default settings of my webcam were on the overexposed side. You can view and alter these settings using `v4l2-ctl` which is part of the `v4l-utils` package:

```
$ v4l2-ctl -d /dev/video0 --list-ctrls
                     brightness 0x00980900 (int)    : min=1 max=255 step=1 default=128 value=128
                       contrast 0x00980901 (int)    : min=1 max=255 step=1 default=128 value=128
                     saturation 0x00980902 (int)    : min=1 max=255 step=1 default=128 value=128
                            hue 0x00980903 (int)    : min=0 max=255 step=1 default=128 value=0
 white_balance_temperature_auto 0x0098090c (bool)   : default=1 value=1
 ```

 I needed to adjust the brightness to 80 and the contrast to 150:

```
$ v4l2-ctl -d /dev/video0 --set-ctrl=brightness=80
$ v4l2-ctl -d /dev/video0 --set-ctrl=contrast=150
```

Once you've found the settings that work for you, you can save them to a bash script and have your desktop environment execute it on login. For example, save this to `~/bin/setup_webcam.sh`:

```bash
#!/usr/bin/bash

v4l2-ctl -d /dev/video0 --set-ctrl=brightness=80
v4l2-ctl -d /dev/video0 --set-ctrl=contrast=150
```

Make the script executable:

```
$ chmod +x ~/bin/setup_webcam.sh
```

Then add it to your autostart applications. For example in XFCE you would go Settings > Session and Startup > Application Autostart and add a new entry to run the command `~/bin/setup_webcam.sh` on login.

## Checking the webcam's mic

Suppose you only wanted to record audio with the webcam's integrated mic. A fast way to do this from the command line is:

```
arecord -f cd test.wav
```

If that file comes out as silent when you play it back, it might just be that the microphone is muted or disabled. The XFCE Pulseaudio Volume Control applet (provided by the `pavucontrol` package) is great for this:

* Go to Pulseaudio Volume Control > Input Devices and:
    * Ensure the mic volume is at 100%;
    * Enable the "set as fallback" button.
* Go to Pulseaudio Volume Control > Configuration and make sure your webcam uses an input profile other than "Off" (you might have to test them one by one if unsure).

Great, but what if you want to use `arecord` to capture the output of your soundcard? In this case you'd need to temporarily disable the webcam microphone. First go to Pulseaudio Volume Control > Configuration and select the "Off" input profile for the webcam, then launch `arecord` as before. 

To switch back to using the microphone, select the correct input profile for the webcam in the Configuration tab and also enable the "set as fallback" button on the Input Devices tab.

## Recording desktop with webcam audio and/or video

I haven't looked too far into this since I think I already found the perfect app for desktop recording: VokoscreenNG, provided by the `vokoscreen` package. With it, you can record your desktop or part of it, in several ways:

* without sound and without webcam
* with sound and without webcam:
    * with sound from the webcam microphone
    * with sound from the sound output device
    * with sound from both
* without sound and with webcam overlay 
* with sound and with webcam overlay:
    * with sound from the webcam microphone
    * with sound from the sound output device
    * with sound from both

...although sound on the recorded video does get to get choppy when using both the webcam mic and the output sound device at the same time.

## Recording from webcam

I thought this was the easiest part of all, but _nooo_, it can't be that simple. As I mentioned in the beginning, I cannot record through the webcam with `cheese` as the video simply freezes. Next, I tried the option of using `vlc`. With VLC you can go to Media > Open Capture Device... and:

* select "Video camera" for capture mode
* specify the video device name (typically, `/dev/video0`)
* take down caching to 0 ms in "show more options"
* select the aspect ratio and the frame rate in "advanced options"

Before you can start recording, however, you need to check "advanced controls" under the View menu (this makes the big red record button show up). Yes, the UI/UX in VLC is just _horrendous_! (In contrast, VokoscreenNG does a really cool job for the interface.)

However, when I record through the webcam with VLC, I get unacceptable frame drops. The video is choppier than chopped salad. So, as ugly as it may seem, I rely on `mencoder`:

```
$ mencoder tv:// -tv driver=v4l2:width=800:height=600:device=/dev/video0:fps=30:forceaudio:alsa:adevice=hw.3,0 -ovc lavc -lavcopts vcodec=mpeg4:vbitrate=1800 -ffourcc xvid -oac mp3lame -lameopts cbr=128 -o output.avi
```

## Conclusion 

In this post I detail the solutions that work best for me and my low-cost webcam, enabling me to make the most of its camera and its integrated mic. Depending on your needs and inclinations, some of the following tools may prove useful:

* `cheese`
* `v4l-utils`
* `pavucontrol`
* `arecord`
* `vokoscreen`
* `vlc`
* `mencoder`

<!-- links -->
[Chinese webcam]: https://www.amazon.fr/dp/B08BNNSTGM
[np]: {% post_url 2021-02-26-autofocus-of-the-logitech-c920-webcam-in-linux %}
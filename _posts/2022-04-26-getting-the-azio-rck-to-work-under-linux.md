---
title: Getting the Azio RCK to work under Linux
date: 2022-04-26 00:00:00 +0100
categories: [Linux, keyboard]
tags: [linux, keyboard, bluetooth]
---

![My Azio Retro Compact Keyboard, Elwood version](/assets/img/posts/azio_rck.jpg){: width="700"}

## Confession time: I hoard mech keyboards

So I have this *thing* (call it a slight obsession) for mechanical keyboards. I'm fascinated by them. My daily driver until now was a 60% keyboard (a [Filco Minila](https://www.diatec.co.jp/en/det.php?prod_c=1320) with MX Browns) but in the past I've used a few IBM Model Ms on a daily basis, especially my cherished "little" [SSK](https://deskthority.net/wiki/IBM_Space_Saving_Keyboard). I also have a Unicomp with  custom Tux keycaps and another Filco Minila with MX Blues, just in case the one with MX Browns breaks down :grin: I caught this virus back in 2014.

Fast forward to 2022: new apartment, new job, new desk (WFH FTW!). Out of the blue, the *best* thrift opportunity that I could have hoped for comes up: somebody was selling their [Azio Retro Compact Keyboard](https://www.aziocorp.com/collections/retro-keyboard-collection/products/rck) in its elwood edition! Meaning exactly what I've been longing to get my hands on for over a year.

It's a gorgeous-looking bluetooth keyboard with Kailh switches that emulate the feel of MX Blues. It can also work via USB and, for what it's worth, it is also backlit if you care for that sort of thing. Without backlight, a single charge lasts for a long time. I've been using it daily for 2 weeks and I'm still at 60% battery. It has the ability to pair to up to three different devices via bluetooth. Which is perfect, because I can use it on my home computer, on my work laptop as well as on my Android phone.

## That ain't working

Except... well, certain manufacturers such as Azio (and also Keychron) seem to purposefully make beautiful keyboards just to spite Linux users.

The first problem I encountered was that I couldn't even detect the keyboard using the `blueman` applet :angry:

The second problem, once I got the keyboard paired, trusted and connected, was that the numlock was enabled by default on Manjaro :roll_eyes:

The third problem was that the function keys were spitting out gibberish :confounded:

## That's the way you do it

Fortunately, solutions exist!

### Detect and connect the keyboard

If the `blueman` applet doesn't see the Azio RCK, or any other bluetooth device for that matter, you will need to launch `bluetoothctl` (provided by the package `bluez-utils` on Arch Linux and Manjaro, or simply by the `bluez` package on Ubuntu):

```
bluetoothctl
Agent registered
[bluetooth]#
```

Now we instruct `bluetoothctl` that it should scan for devices as they become available:

```
[bluetooth]# scan on
```

At this point, the keyboard should be in pairing mode. For the Azio RCK, first choose which device to pair among the possible three by pressing `Fn` + `1`/`2`/`3`. Then enter pairing mode by pressing and *holding* `Fn`+`Ins`. As the device becomes available, `bluetoothctl` will signal this by showing the device address (it's like a MAC address).

Using the `12:34:56:78:90:AB` placeholder, you can now pair with the device, trust it and connect to it:

```
[bluetooth]# pair 12:34:56:78:90:AB
[bluetooth]# trust 12:34:56:78:90:AB
[bluetooth]# connect 12:34:56:78:90:AB
```

Now you're good to go.

### Turn off the numeric lock

In case only the right part of the keyboard seems to work but it only outputs numbers, it means the numlock is on. Turn it off by installing the `numlockx` package and by running:

```
numlockx off
```

### Enable function keys

Earlier in this post I mentioned Keychron because the solution comes from [somebody](https://venthur.de/2021-04-30-keychron-c1-on-linux.html) having experienced the same problem with their keyboards on Debian.

The solution is to write `2` in `/sys/module/hid_apple/parameters/fnmode`, as root:

```bash
echo 2 | sudo tee /sys/module/hid_apple/parameters/fnmode
```

This file describes how `fn` keys behave: more info [here](https://help.ubuntu.com/community/AppleKeyboard#Change_Function_Key_behavior).

If this solves the problem, the change can be made permanent by creating `/etc/modprobe.d/hid_apple.conf` with the following contents:

```
options hid_apple fnmode=2
```

Then run

```bash
sudo update-initramfs -u -k all
```

## TL;DR

Yes, the Azio RCK works under Linux with a bit of tinkering :relieved:
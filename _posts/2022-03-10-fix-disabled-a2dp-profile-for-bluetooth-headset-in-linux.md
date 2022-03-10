---
title: Fix disabled A2DP profile for bluetooth headset in Linux
date: 2022-03-10 00:00:00 +0100
categories: [Linux, audio]
tags: [linux, audio, bluetooth]
---

AudioPulse has died on me. RIP PulseAudio. I can no longer select the A2DP profile for my bluetooth headset (Sony WH-1000XM4) under Manjaro Linux. I tried everything: switching between kernels (5.15 through 5.17), the [ArchLinux wiki][] and demon worshipping among others, but nothing helped.

"No A2DP profile" means that everything sounds as if coming out of a tin can (also known as HSP/HFP, i.e. headset/hands-free profile).

## Replace PulseAudio with PipeWire

What ultimately worked for me was to replace PulseAudio with PipeWire. When installing the metapackage `manjaro-pipewire`, the installer will automatically flag PulseAudio packages as conflicting with the PipeWire equivalents. In other words, if everything works smoothly, the following command should replace PulseAudio with PipeWire:

```
sudo pacman -S manjaro-pipewire
```

If some unresolved dependencies remain, remove them with `pacman -R` first.

Finally, reboot:

```
sudo reboot
```

## Bluetooth headset profile cleanup

If everything works well, the bluetooth device corresponding to your headset should have a selectable A2DP profile:

![Bluetooth audio profile](/assets/img/posts/bluetooth_audio_profile.png){: width="700"}

If this is not yet the case, there's a final cleanup step that may be required:

1. Unpair the headset
1. Remove old settings:
```
sudo rm -fr /var/lib/bluetooth
```
1. Re-pair the headset

## Automatically switch to the bluetooth headset

In a [previous post][pp], we've seen how to automatically switch to the bluetooth headset once it's on. That was valid for PulseAudio. For PipeWire it's a bit different. To achieve this, edit the file `/usr/share/pipewire/pipewire-pulse.conf` and, under `content.exec`, uncomment the following line:

```
{ path = "pactl" args = "load-module module-switch-on-connect" }
```

After this step, if restarting PipeWire is not sufficient, just reboot a last time and everything should be well. The bluetooth headset should be switched to when it is turned on.

<!-- links -->

[ArchLinux wiki]: https://wiki.archlinux.org/title/bluetooth_headset#A2DP_not_working_with_PulseAudio
[pp]: {% post_url 2021-07-14-connect-bluetooth-headset-automatically-linux %}
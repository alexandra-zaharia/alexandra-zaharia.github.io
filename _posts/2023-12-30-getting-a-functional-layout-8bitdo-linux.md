---
title: Getting a functional layout in Linux for the 8BitDo Retro Mechanical Keyboard
date: 2023-12-30 00:00:00 +0100
categories: [Linux, keyboard]
tags: [linux, keyboard, bluetooth]
---

![My 8BitDo Retro Mechanical Keyboard, N edition](/assets/img/posts/8bitdo.jpg){: width="700"}

## Confession time: I _still_ hoard mech keyboards

It's been more than a year since I last posted here (things got... complicated). Incidentally, the last post was about getting the [Azio Retro Compact Keyboard][] to work properly under Linux. Well, I *still* have this thing for mechanical keyboards, only now I have one extra keyboard, another one on the way and yet another one that I'm dead set on buying. 

This post is about the most recent addition to my collection: the [8BitDo Retro Mechanical Keyboard][], N edition.

It's awesome! It has great looks if you're into the retro gaming aesthetic, great build quality, PBT key caps, Kailh box white switches (**so** much better than blue switches!), volume knob, old-style power LED, magnetic USB dongle, extra controller pad ("dual pad") with two big fat red buttons. Connectivity-wise, it can be used wired, with a 2.4 GHz USB dongle, or via bluetooth. Also, the price tag is insane (in a good way)!

What I don't like about it:
* The extra pad with the two big buttons has to be physically wired to the keyboard (it is using a 3.5 mm jack)
* The software :roll_eyes:
* Some macros don't work properly when the keyboard is connected via bluetooth
* Unlike the Azio RCK and other keyboards, it can only be paired to a single device at a time in bluetooth mode

That being said, I still think it's awesome and would totally buy it again :grin:

## The plan

The 8BitDo Retro Mechanical Keyboard comes with two extra keys labeled "A" and "B", in between the right-hand Alt and Ctrl keys. Neither them, nor the big buttons on the dual pad emit any keycodes out of the box (I used `xev` to check that).

What I wanted to do was:
* Get the keyboard `B` button to be the Menu key (the context menu that you get on some keyboards).
* Get the keyboard `A` button to be Play/Pause (so that it can work with Spotify for example).
* Get the screen lock macro assigned to the big `B` on the dual pad (it's `Ctrl + Alt + L` for me).
* Get my user password followed by a line feed assigned to the big `A` on the dual pad.
* Get the volume knob "fixed" because for some strange reason turning it to the right to increase the volume wouldn't do anything.

## The software

The 8BitDo Ultimate Software V2 is Windows only. I couldn't get it to run with wine or lutris, so I ultimately installed it in a VirtualBox Windows VM.

Oddly enough, I couldn't get it to run inside a Windows 10 VM with USB passthrough, even though it's supposed to work with Windows 10. So I ended up installing Windows 11 in VirtualBox (with some [registry edits][] during the install to bypass the hardware checks, otherwise it wouldn't install for me).

## Key mappings

After a firmware update I could finally start mapping the keys under a new profile:

* For the Super Button 1 (keyboard "A") I chose `Shortcut` then `Play/Pause`. This works great with either Youtube or Spotify even if it's playing on another screen, which is great.
* For the Super Button 2 (keyboard "B") I chose `Shortcut` then `Mail` (more on this later).

For the dual pad, I spent some time realizing that if you only have a single dual pad, it is only recognized if plugged into the `X` port on the keyboard. You see, the keyboard accepts up to 4 dual pads through its `A`, `B`, `X` and `Y` ports, but if you only have one, then only port `X` specifically seems to work. 

For both dual pad buttons I assigned macros (one of them would lock my screen and the other one would unlock it by typing in my user password and hitting enter).

## Some Linux-specific adjustments

### Remapping a key

Let's get back to that weird `Mail` mapping. Remember the extra buttons have no keycode assigned to them, or nothing that `xev` can see anyway. Moreover, I couldn't just map the `Menu` key directly to one of the buttons, because the software didn't offer this key as a mapping option :shrug: So I figured I'd just map to something I would never use, such as the `Mail` shortcut that you get on some "multimedia" keyboards. Such a seemingly useless button mapping turns out to be useful in the end because it can be changed later via `xmodmap`.

I first looked at the keycodes I have by running:

```bash
xmodmap -pke > ~/.xmod
```

In the generated `~/.xmod` file, I found the keycode `163` for `XF86Mail`. To assign the `Menu` key to this otherwise useless keycode, I added this to my `~/.Xmodmap`:

```bash
keycode 163 = Menu
```

And then I loaded it with:

```bash
xmodmap ~/.Xmodmap
```

Problem solved: now I get the `Menu` key whenever I hit the red `B` button on the keyboard.

### Fixing the knob for volume up 

For some strange reason, turning the volume knob to decrease the volume worked as expected, but turning the volume up wouldn't do anything. What I settled on doing was to increase the volume through `pactl` when the volume knob is turned right, and have the whole thing handled by the Xfce keyboard shortcuts. The one I added was running this command for the `XF86AudioRaiseVolume` "key":

```
pactl set-sink-volume @DEFAULT_SINK@ +5%
```

## Random failures

### Factory reset

I thought I got permanently locked out of the software because it would yell at me in Chinese and not even allow me to check the existing profiles. I have no idea why that happened. I guess the software didn't like my dual pad macros because it removed them without me asking it to do so, so I basically ended up with a useless dual pad and a non-working software. 

Fortunately, there is a factory reset,  although it's not documented anywhere. To reset to factory settings, just hold down all 3 round buttons together (pair, fast mapping and profile) for a few seconds. Then there's some blinking going on as your profile is erased and the software should work again if it ever gets stuck like it did for me.

### Sucky bluetooth

Another pain point is the way the macros work (or rather don't work) via bluetooth. The short one to lock my session (`Ctrl + Alt + L`) is alright, but the other one to type in my password is just broken because it repeats certain keystrokes. It works flawlessly wired and via the 2.4 GHz dongle though so I assume it has something to do with the delays between keystrokes not working well with bluetooth delays. And we all know that bluetooth just sucks, period. Sure, I _could_ try to edit the delays between keystrokes with the software, but that's what got me in trouble as described above, being locked out of the software. I'll just call it a day and accept a slow macro for the time being :grin:

## Conclusion

All in all, the 8BitDo is a very nice keyboard with a decent extent of customisability, but the software kind of sucks. I'm glad I got it though! :nerd_face:

<!-- links -->
[Azio Retro Compact Keyboard]: {% post_url 2022-04-26-getting-the-azio-rck-to-work-under-linux %}
[8BitDo Retro Mechanical Keyboard]: https://www.8bitdo.com/retro-mechanical-keyboard
[registry edits]: https://www.minitool.com/news/this-pc-cant-run-windows-11-on-virtualbox.html

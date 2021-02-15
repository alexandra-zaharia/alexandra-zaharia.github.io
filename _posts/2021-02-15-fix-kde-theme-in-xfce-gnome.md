---
title: Fix the look of KDE apps in other desktop environments
date: 2021-02-15 19:30:00 +0100
categories: [Linux, desktop environment]
tags: [xfce, kde, qt5]
---

## The problem

Ever re-installed your OS but kept your `/home` partition and got to experience some weird visual artifacts? If your desktop environment is XFCE for instance and your KDE apps (such as `okular`, `konsole`, `spectacle` or `gwenview`) look bad, then you'd normally solve this with [Kvantum Manager][]. Kvantum is used for styling Qt apps. It worked great out of the box for the Manjaro clean installs I've done so far, but the same cannot be said about the latest (re-) install where I just kept my `/home` partition and installed Manjaro onto a brand new SSD.

## The solution

Again, out of the box, Kvantum Manager works great... until it doesn't. I might have forgotten to install a particular Qt theme. Instead of tracking down that particular package, an alternative solution is to install `qt5ct`. As explained on this Arch Linux [wiki page][], `qt5ct` provides another way to style KDE apps from within a desktop environment other than KDE. So this should work for XFCE, as well as Gnome and what have you.

Just install it:

```bash
sudo pacman -S qt5ct
```

Next, you need to make an environment variable available globally to specify that it is `qt5ct` that should be used as the Qt platform abstraction. This can be achieved by adding the following line to `/etc/profile`:

```bash
export QT_QPA_PLATFORMTHEME=qt5ct
```

Now launch `qt5ct` and select the desired theme.

Log out and log in again and your KDE apps should now be better integrated visually into your non-KDE desktop environment.

<!-- links -->
[Kvantum Manager]: https://store.kde.org/p/1005410/
[wiki page]: https://wiki.archlinux.org/index.php/qt#Configuration_of_Qt5_apps_under_environments_other_than_KDE_Plasma

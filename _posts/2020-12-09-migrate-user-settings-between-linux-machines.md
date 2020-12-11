---
title: How to migrate user settings and data between Linux machines on a LAN
date: 2020-12-09 20:26:00 +0100
categories: [Linux, command line]
tags: [linux, manjaro, migration]
---

Ever had to set up a new Linux machine (for example your work laptop), and wanted to import the exact same settings that you use on your main (or home) machine? Maybe some data as well?

Some steps of this guide are Manjaro Linux (and, by extension, Arch Linux) specific, but some of the general ideas could be transplanted to other distros as well. Your mileage may vary with respect to  my specific setup and needs. Some of the steps below assume the two machines are in a local area network (LAN).

Here is what we will be covering:

1. Using the same mouse and keyboard to control the two machines, provided they are conveniently placed on a desk side by side.
2. Setting up OpenSSH server on the new machine to be able to copy files from the "old" one.
3. Making a list of packages installed on the new machine that we also need on the new machine. This can save a lot of typing!
4. Copying user settings from the old to the new machine.

In this post, let's say we want to migrate settings and files from a machine with hostname `desktop` (LAN IP 192.168.1.28) to another machine with hostname `laptop` (LAN IP 192.168.1.24). 

## Share mouse and keyboard

First things first. If we're talking about a LAN and the two machines are side by side on a single desk, it's handy to use the "main" one's keyboard and mouse to control both. For this you can use `synergy`. After installing Synergy on both machines, the steps are as follows:
1. Make each machine know the other's name. It's not mandatory but it's more user-friendly to connect to `desktop` than to 192.168.1.28. To do this, edit each machine's `/etc/hosts` file by adding a line with the other's machine LAN IP followed by its hostname. For example, on the main `desktop` machine (IP 192.168.1.28) I would add `192.168.1.24 laptop` to `/etc/hosts`. To get the LAN IP, examine the output of `ip addr` (or `ifconfig` for other Linux distros).
2. Edit `/etc/synergy.conf` on the main (`desktop`) machine. You need to tell it where the other machine is, physically. I have my laptop to the left of my desktop, so my `/etc/synergy.conf` file looks like this:

    ```
    section: screens
            desktop:
            laptop:
    end

    section: links
            desktop:
                    left = laptop
            laptop:
                    right = desktop
    end
    ```
3. Start the Synergy server on the main machine (`desktop` in my case) by running `synergys`. This is the machine whose mouse and keyboard will control the other.
4. Start the Synergy client on the new machine (`laptop` in my case) by running `synergyc desktop`. This means that this machine will connect its Synergy client to the Synergy server running on `dekstop` (where the Synergy server must already be running).

That's all. Now you can use only one mouse and keyboard for both machines.

## Copy files to new machine

Once keyboard and mouse input are available through your main device on the new machine, you can start copying files comfortably (i.e. without being endangered by scoliosis). OpenSSH server needs to be installed on the new machine first, though. In Manjaro and Arch Linux:

```bash
alex@laptop$ sudo pacman -S openssh
```

Now to configure it, edit `/etc/ssh/sshd_config` on the new machine (`laptop`) as follows (replace the `ListenAddress` with the LAN IP of your new machine):

```
ListenAddress 192.168.1.24
ListenAddress 127.0.0.1
PermitRootLogin no
MaxAuthTries 3
```

I also like to enable logging such that I can check my SSH logs with `journalctl -u sshd`. For this, edit `/etc/ssh/sshd_config` to add the following (or uncomment these lines under the Logging section):

```
SyslogFacility AUTH
LogLevel INFO
```

Then you need to start OpenSSH server on the new machine:

```bash
alex@laptop$ sudo systemctl start sshd.service
```

The first time you connect to the new machine, you will be asked to [check the host fingerprint][ssh_post]. 

To copy files from the main to the new machine, use `scp`. 

```bash
alex@desktop$ scp ~/doc.txt alex@laptop:~
```

If you want to copy a directory over to the new machine, just use the `-r` switch with `scp`:

```bash
alex@desktop$ scp -r wallpapers/ alex@laptop:~/Pictures
```

## Install the same packages on the new machine

Before starting any package install, it is best to update the mirror list with the fastest mirrors. On the new machine, run:

```bash
alex@laptop$ sudo pacman-mirrors -f 10
```

This will determine the 10 fastest mirrors and update `/etc/pacman.d/mirrorlist`.

Now generate the list of installed software on the old machine and copy it over to the new machine:

```bash
alex@desktop$ pacman -Qqen > pkglist.txt
alex@desktop$ scp pkglist.txt alex@laptop:~
```

Be sure to remove what you don't need, for example NVidia drivers if the new machine does not have an NVidia graphics card. (In my case, the original list contained 399 packages, of which I only wanted to install 61 to the new machine.)

On the new machine, install the packages listed in `~/pkglist.txt`:

```bash
alex@laptop$ sudo pacman -S --needed - < ~/pkglist.txt 
```

If, by chance, you have installed KDE's Baloo file indexer as a dependency, do yourself a favor and disable that monstrosity:

```bash
alex@laptop$ balooctl disable
```

## Copy user settings over to the new machine

If you use the same desktop environment on both machines (this example assumes we're using XFCE), you can attempt the following method. **It is important to replace the files on the new machine from an actual TTY, NOT from the graphic session**. First, back up the destination folders on the new machine:

```bash
alex@laptop$ cp -fr ~/.config/xfce4/ ~/.config/xfce4.bck
alex@laptop$ cp -fr ~/.local/share/xfce4 ~/.local/share/xfce4.bck
```

Then, prepare a temporary folder for the XFCE configuration files. You don't want to be copying them while XFCE is up and running: 

```bash
alex@laptop$ mkdir ~/tmp/
alex@laptop$ mkdir ~/tmp/.config ~/tmp/.local/share
```

Then, copy them over from the main machine to the temporary folders on the new machine:

```bash
alex@desktop$ scp -r ~/.config/xfce4/ alex@laptop:~/tmp/.config
alex@desktop$ scp -r ~/.local/share/xfce4/ alex@laptop:~/tmp/.local/share/
```

**Now log out from the new machine, then open a TTY (Ctrl + Alt + F1, Ctrl + Alt + F2, etc.).** From the TTY, first remove the default XFCE configuration, then move the new one in its place. Finally, restart (do a full reboot):

```bash
alex@laptop$ rm -fr ~/.config/xfce4
alex@laptop$ rm -fr ~/.local/share/xfce4
alex@laptop$ mv ~/tmp/.config/xfce4 ~/.config
alex@laptop$ mv ~/tmp/.local/share/xfce4 ~/.local/share
alex@laptop$ sudo reboot
```

After reboot, your old settings should be applied to the new machine (but keep in mind that some adjustments might be necessary). If everything is FUBAR-ed, then just log out, switch to TTY and rename the two `xfce4.bck` directories to `xfce4`.

## Conclusion

This guide for migrating user data and settings might be useful for quickly migrating the most important stuff between two Arch Linux-like distros. Happy set up!

<!-- links -->
[ssh_post]: {% post_url 2020-09-28-how-to-check-the-ssh-host-key-fingerprint %}

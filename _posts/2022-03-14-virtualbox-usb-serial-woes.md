---
title: VirtualBox USB and serial woes
date: 2022-03-14 00:00:00 +0100
categories: [Linux, virtualization]
tags: [linux, manjaro, vm]
---

Enabling USB and serial devices in a VirtualBox VM is not straightforward. For reference, the host is a Manjaro Linux machine and the guest OS is a Ubuntu VM.

## Enable USB 2.0 and USB 3.0 controllers

In order for VirtualBox to propose USB 2.0 and 3.0 controllers for the VM, install the `virtualbox-ext-oracle` extension package.

## Configure serial ports on the VM

If the device that the guest VM needs to access is a serial device, go to the VM's settings under Serial Ports > Enable Serial Port and choose **Host Device** as Port Mode with the proper path/address (e.g. `/dev/ttyACM0`).

## Add your user to the vboxusers group

If your current user does not belong to the `vboxusers` group, VirtualBox will be unable to show the USB devices currently in use. Run the following command:

```
sudo usermod -a -G vboxusers $USER
```

Logout or restart.

## Add USB device filters

From the VM's settings under USB, add a USB device filter for each USB (or serial) device that you want to render accessible to the guest VM. It should look something like this:

![USB device filter](/assets/img/posts/virtualbox_usb_device_filter.png){: width="700"}

## Done

Now start the VM. The USB and/or serial devices added as USB device filters in the VM's settings will now be accessible in the VM.
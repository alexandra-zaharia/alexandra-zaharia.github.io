---
title: How to manually modify a .deb package
date: 2008-06-26 00:00:00 +0100
categories: [Linux, command line]
tags: [linux, deb]
math: true
---

[I originally published this post on [Blogspot](https://plug-and-pray.blogspot.com/2008/06/manually-modifying-deb-package.html).]

Every now and then you might need to tweak a .deb package for testing purposes. For instance you just want to change the behavior of the _postinst_ script. Here are the steps to extract, modify and repackage the .deb package.

**Step 1.** Unpack with `ar`:

```bash
ar x package.deb
```

You obtain three files:

* `debian-binary`: ASCII text
* `control.tar.gz`: gzip compressed data. This archive contains the files that actually get installed on the system (in `/etc`, `/opt` or `/usr` for example).
* `data.tar.gz`: gzip compressed data. This archive usually contains:
  * `control`: ASCII text
  * `md5sums`: contains the MD5 sums for the files in `data.tar.gz`
  * `postinst`: a script that is executed after installing the package
  * `postrm`: a script that is executed after removing the package
  * `preinst`: a script that is executed before installing the package
  * `prerm`: a script that is executed before removing the package

> Some packages do not have pre/post install/removal scripts.

**Step 2.** Unpack the archive(s) depending on which files you need to modify. To unpack both archives, run:

```bash
mkdir extras-control extras-data
tar -C extras-control/ -zxf control.tar.gz
tar -C extras-data/ -zxf data.tar.gz
```

**Step 3.** Modify the file(s) you need to change. If you modified files that are included in `data.tar.gz`, you need to update their MD5 sum in `extras-control/md5sums`.

**Step 4.** Repack `control.tar.gz` and/or `data.tar.gz` and remove the temporary directories:

```bash
cd extras-control
tar cfz control.tar.gz *
mv control.tar.gz ..
cd ../extras-data
tar cfz data.tar.gz *
mv data.tar.gz ..
cd ..
rm -fr extras*
```

**Step 5.** Repackage the .deb package:

```bash
ar r new_package.deb debian-binary control.tar.gz data.tar.gz
```

> **Note:** I haven't had the chance to test this recently, but it would appear that the order of files in the last step (step 5) is important.
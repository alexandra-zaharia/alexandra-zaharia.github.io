---
title: How to split files in Linux from the command line
date: 2009-11-25 00:00:00 +0100
categories: [Linux, command line]
tags: [linux, split]
math: true
---

[I originally published this post on [Blogspot](https://plug-and-pray.blogspot.com/2009/11/how-to-split-large-files-in-linux-and.html).]

It can be useful to split large files, or even smaller files and ensure all the resulting volumes have the same size. For this we will be using `split`.

## Splitting files

We can split a larger file into smaller ones like this:

```bash
split -b 100M file
```

The previous command splits `file` into several 100 Mb volumes, called by default `xaa`, `xab`, `xac` and so on. These default names may be prefixed by a pattern:

```bash
split -b 100k file pattern_
```

The previous command splits `file` into several 100 Kb volumes, called `pattern_aa`, `pattern_ab`, `pattern_ac` and so on. If we want digits instead of letters, we can use the `-d` flag:

```bash
split -db 1G file pattern.
```

The previous command splits `file` into several 1 Gb volumes, called `pattern.00`, `pattern.01`, `pattern.02` and so on.

## Joining files

To join the volumes, we can `cat` the sorted file names and redirect them to an output file:

```bash
cat `echo pattern.* | sort` > new_file
```

Both the original `file` and `new_file` have the same MD5 sum; they are identical.
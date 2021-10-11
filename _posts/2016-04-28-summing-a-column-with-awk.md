---
title: Summing a column with awk
date: 2016-04-28 00:00:00 +0100
categories: [Linux, command line]
tags: [linux, awk]
---

Suppose you want to sum values on the _n_<sup>th</sup> column in a file. Here is how to do this using awk:

```bash
awk '{ sum += $<COL> } END { print sum }' <FILE>
```

Replace `<COL>` with the index of the column (the first one has index 1 and the last column can be referred to as `NF`). Replace `<FILE>` with the file name.

Your file might look something like this:

```
 1  2  3
 4  5  6
 7  8  9
10 11 12
```

Here is the output for this file:

```bash
$ awk '{ sum += $1 } END { print sum }' data.txt 
22
$ awk '{ sum += $2 } END { print sum }' data.txt 
26
$ awk '{ sum += $3 } END { print sum }' data.txt 
30
$ awk '{ sum += $NF } END { print sum }' data.txt
30
```
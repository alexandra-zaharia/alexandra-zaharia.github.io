---
title: How to detect modified files across two directories
date: 2022-03-29 00:00:00 +0100
categories: [Linux, command line]
tags: [linux, md5, bash]
---

Suppose you have two directories `old` and `new` containing the same files, but some of them have changed in the `new` directory. Further suppose that source control is not used :scream: This post explains how to detect the modified files based on their MD5 sums through a simple bash script.

For this example, suppose the file types we are interested in are Python scripts (`.py` files). Note that we define the variables `NEW_DIR` and `OLD_DIR` (representing the paths to the `new` and the `old` directories, respectively) using absolute paths (i.e. paths that start from the root of the filesystem).

```bash
#!/usr/bin/bash

NEW_DIR="/path/to/new"
OLD_DIR="/path/to/old"

cd $NEW_DIR
files=`find . -name "*.py"`
for file in $files ; do
    filename=`echo $file | sed 's/\.\///g'`
    orig="${OLD_DIR}/${filename}"
    md5_orig=`md5sum $orig | awk '{ print $1 }'`
    md5_new=`md5sum $file | awk '{ print $1 }'`
    if [[ "${md5_orig}" != "${md5_new}" ]] ; then
        echo $file $md5_orig $md5_new
    fi
done
```

In lines 6-7, we search for all the files ending in `.py` in the `$NEW_DIR`. For each matching file, we transform its `filename` into `orig`, i.e. the corresponding path and file name in the `$OLD_DIR` (lines 9-10). We then compute the MD5 sum of the two files (lines 11-12) and display only files where the MD5 sums differ (lines 13-15).
---
title: Definitely not the definitive guide to Mercurial merge
date: 2020-12-12 00:00:00 +0100
categories: [Tools, Mercurial]
tags: [mercurial, version control]
---

## Oh no, not another how to!
Sure, there is a ton of documentation out there when it comes to version control. Take Mercurial for instance, there are [excellent guides][guide] out there on merging changes.

But sometimes I just need a very simple workflow to merge changes in the same repo between two machines. Enter bad memory into the game and the only solution is this kind of quick and dirty how to.

## The calm before the storm

My `~/.hgrc` contains the option to let me merge manually instead of using third-party tools. This way, conflicts are signaled between `<<<` and `>>>` symbols, just like in Git. This is the option in question:

```
[ui]
merge=internal:merge
```

## All hell breaks loose

Now just merrily do your `pull` and... uh-oh:

```
searching for changes
adding changesets
adding manifests
adding file changes
added 1 changesets with 1 changes to 1 files (+1 heads)
new changesets a3256593884f (1 drafts)
(run 'hg heads' to see heads, 'hg merge' to merge)
```

Now the repo has two [heads][] that can be viewed with `hg heads`. From there, select the head to merge with. For example, suppose the tip of the working copy is changeset 8, and the one pulled from remote is 9. In this case, you'd merge with revision 9:

```
hg merge -r 9
```

In return, you might get something like this if there are any conflicts:

```
merging my_file
warning: conflicts while merging my_file! (edit, then use 'hg resolve --mark')
0 files updated, 0 files merged, 0 files removed, 1 files unresolved
use 'hg resolve' to retry unresolved file merges or 'hg merge --abort' to abandon
```

Open the troublesome `my_file` and resolve the conflict. Let's say it looks like this:

```
This is my file
<<<<<<< working copy
This was edited locally
=======
This was edited in upstream
>>>>>>> merge rev
```

Suppose we want to discard the local change. Then we'd edit the file above like this:

```
This is my file
This was edited in upstream
```

## After rain comes sunshine

Now we tell `hg` that everything is fine:

```
hg resolve --mark
```

We commit the merge:

```
hg commit -m "merged"
```

And we're done!

<!-- links -->
[guide]: https://book.mercurial-scm.org/read/merge.html
[heads]: https://www.mercurial-scm.org/wiki/Head

---
title: How to squash two non-consecutive commits in Mercurial
date: 2020-12-11 18:15:00 +0100
categories: [Tools, Mercurial]
tags: [mercurial, version control]
---

## The problem

Suppose you want to combine two commits in Mercurial. However, they are not consecutive. What to do? `histedit` to the rescue!

Suppose `hg log` shows that your last 4 revisions are the following:

```
3: more stuff on Y
2: stuff on X
1: stuff on Y
0: stuff on Z
```

And suppose you want to squash what you commited on Y into a single revision, like this:

```
2: stuff + more stuff on Y
1: stuff on X
0: stuff on Z
```

## Reorder revisions

The first step is to reorder the revision history in order to make the revisions to be squashed become consecutive (`stuff on Y` and `more stuff on Y`):

```
hg histedit
```

Select the revision you want to move (`1: stuff on Y`) and move it _down_ (press capital `J`). Why down? Well, `hg log` is reverse chronological so the latest revision on top, while `hg histedit` is chronological so the latest revision is at the bottom. Confusing, I know. Just commit (press `c`) and now your `hg log` should show the reordered revisions:

```
3: more stuff on Y
2: stuff on Y
1: stuff on X
0: stuff on Z
```

## Squash the two commits

Since the revisions on Y are now consecutive, we can finally "fold" them (they might have called it "squash", it would have been more intuitive). However, we must tell Mercurial from which revision to start: we want revision number 2 in the log above, since it is the one on top of which the other one will be "folded":

```
hg histedit 2
```

Then select revision 3 (`more stuff on Y`) and select "fold" (press `f`). Commit (press `c`) and edit the commit message (e.g., `stuff + more stuff on Y`) and you should now have the `hg log` that you wanted all along:

```
2: stuff + more stuff on Y
1: stuff on X
0: stuff on Z
```

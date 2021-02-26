---
title: Mercurial branch how-to
date: 2021-02-17 23:30:00 +0100
categories: [Tools, Mercurial]
tags: [mercurial, version control]
---

This is a quick how-to on *named* branches in Mercurial. Be sure to check out the official documentation on branches on the [Mercurial wiki][hg wiki], the visual explanation on the different types of branches by [Steve Losh][losh] and the named branches tutorial by [Mark Heath][named] first.

So now that you concluded that clones and bookmarks are not the solution to your branching problem, you want to create a named branch. For the purpose of this example, assume the new branch is called `dev`.

## Creating a named branch

If you want to branch from a previous changeset, update to that revision number first. Suppose you're at revision 10 and you want to branch from revision 5:

```
hg up 5
```

Now you're still on the `default` branch but you just went back in time from revision 10 to revision 5. 

It's time to create the new `dev` branch. It will be created from the current changeset if you haven't used `hg up` as above:

```
hg branch dev
``` 

When a branch is created, it automatically becomes the current one. You can check this if you want with `hg branch`.

The list of existing branches is viewable with `hg branches`. However, the `dev` branch will not be listed there as long as there are no commits to it. So change something, then commit. Now `hg log` shows your new commit as being on the `dev` branch.

## Switching branches

If you want to go back to the default branch, use:

```
hg up default
```

Change something there, commit, then come back to the `dev` branch:

```
hg up dev
```

What happens if you want to switch branches when your current branch contains **uncommitted changes**? Mercurial does not allow this. You can either use the `shelve` extension or create a patch. I'm going with the patch solution. If we're on the `dev` branch with uncommitted changes, here is how to create a patch before switching to the `default` branch:

```
hg diff > dev.patch
```

In order to switch, we must tell Mercurial to discard uncommitted changes (`-C`), which is OK since we've already saved them as `dev.patch`:

```
hg up -C default
```

When going back from the `default` to the `dev` branch, we need to import the uncommitted changes:

```
hg up dev
hg import --no-commit dev.patch
```

## Viewing the differences

To view the differences between two branches `branch1` and `branch2`, use:

```
hg diff -r branch1:branch2
```

If we only have two branches (`default` and `dev` in our example), we can exclude the current branch. Suppose we're on the `dev` branch and we want to see the differences with respect to `default`: 

```
hg diff -r default
```

If we are only interested in differences between two branches for a particular `file`:

```
hg diff -r branch1:branch2 file
```

For our example, we are on the `dev` branch and we want to see how `file` differs with respect to the same file in the `default` branch:

```
hg diff -r default file
```

That's all! Happy branching! 

<!-- links -->
[hg wiki]: https://www.mercurial-scm.org/wiki/Branch
[losh]: https://stevelosh.com/blog/2009/08/a-guide-to-branching-in-mercurial/
[named]: https://markheath.net/post/using-named-branches-in-mercurial

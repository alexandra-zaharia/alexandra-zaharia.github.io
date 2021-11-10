---
title: Merge Mercurial branch into default
date: 2021-11-10 00:00:00 +0100
categories: [Tools, Mercurial]
tags: [mercurial, version control]
---

In a [previous post][pp] I wrote a quick recipe for doing a merge in Mercurial when pulling from the remote repository.

This post is another quick Mercurial merge recipe, but this time for merging a development branch (let's call it `dev`) into the `default` branch.

## First things first

Save yourself some heart ache by ensuring there's nothing left to commit to the `default` branch before creating the new `dev` branch. And absolutely do a `hg pull` from the remote repository beforehand! Personal experience... :fearful:

## Branch away

So you finally created the branch:

```
hg branch dev
```

And happily started `dev`ing. Some 9,000 hours later, the new branch is squeaky clean and you want to merge it into `default`. Same as before, make sure there's nothing left to commit in `dev`. Check this with `hg status`.

## The actual merge

Switch back to the `default` branch:

```
hg up default
```

Do the merge and commit:

```
hg merge dev
hg ci -m "merge with branch dev"
```

## Close the branch

If you think there's no need to keep the `dev` branch open any longer, you can close it:

```
hg up dev
hg ci -m "close branch dev" --close-branch
```


<!-- links -->
[pp]: {% post_url 2020-12-12-simple-mercurial-merge %}
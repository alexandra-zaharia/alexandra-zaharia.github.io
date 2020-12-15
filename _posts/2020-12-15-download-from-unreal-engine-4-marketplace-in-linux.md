---
title: Download UE4 assets from the Epic Games Marketplace using Linux
date: 2020-12-15 21:00:00 +0100
categories: [Unreal Engine 4, assets]
tags: [linux, ue4]
---

It is an extremely frustrating experience to have an Epic Games account (for Unreal Engine 4) but not be able to download assets from the Marketplace because... Linux. :angry:

## The problem with Linux and the UE4 Marketplace

The Unreal Engine itself is technically supported on Linux: while you cannot download a pre-packaged version (unless your favorite distro maintainers are really cool), you _can_ link your GitHub account to your Epic Games account in order to gain read access to the Unreal Engine 4 repository. From there, you can clone the UE4 release that you need and compile it from source. 

But figure this: your Epic Games account also gives you access to the Unreal Engine Marketplace. You can use your browser to connect, browse the marketplace and get free assets or buy non-free ones.  Therein lies the problem, you see. You can only "link" them to your account. You cannot download those assets on Linux using the browser. The download button tries to open the UE4 Editor and since the Linux version does not come with the marketplace bundled, you won't be able to download those assets. 

You can build your UE4 projects on Linux. You just can't get assets for them from the Epic Games Marketplace... on Linux. 

## There is a solution

And here is where [Lutris][] comes to the rescue! Lutris is an open-source platform for playing games on Linux. Among the many games that you can download in their Windows version to install in Linux through Wine (provided that you have them in your Steam or GOG libraries), you can also download certain game store clients such as Origin or Epic Games. 

## Epic Games launcher through Lutris

Install Lutris and launch it from a **Python 3.8+ environment**. Alternatively, ensure the default Python version is 3.8+. This is very important, because if you use an older Python version the install will seemingly complete but the Epic Games launcher will not exist. 

In Lutris, search for "epic games" and install the Epic Games Store. Once finished, check that you have your `EpicGamesLauncher.exe` executable in your Epic Games install path. For this example, let's suppose you installed the Epic Games Store in `~/Programs/epic-games-store`. Then make sure that `~/Programs/epic-games-store/drive_c/Program\ Files\ \(x86\)/Epic\ Games/Launcher/Portal/Binaries/Win32/EpicGamesLauncher.exe` exists.

You don't need to install the Windows version of the Unreal Engine through Wine/Lutris in order to download assets, you just need a UE4 project. And what better way to get your assets imported directly into your Linux projects than to symlink your `Unreal Projects` directory to the virtual Wine file system?

Here is how to do it. Suppose you store your Linux UE4 projects in `~/Documents/Unreal Projects`. You just need to create a link from your hypothetical Windows `My Documents` to your actual `Unreal Projects` folder in Linux:

```bash
ln -sf \
    ~/Documents/Unreal\ Projects \
    ~/Programs/epic-games-store/drive_c/users/YOUR_USERNAME/My\ Documents/Unreal\ Projects
```

(Don't forget to replace `YOUR_USERNAME` with your user name and to backslash-escape spaces in path names.)

Completely close the Epic Games Store and Lutris and launch them again. Connect to your Epic Games account, then go to Unreal Engine > Library. Your downloadable assets are in your Vault (see below):

![Epic Games launcher running on Linux](/assets/img/posts/epic_games_marketplace_linux.png)

Now select an asset from the Vault and click "Add to project". A new window will open asking you to which project to add the asset, but your projects might not be visible. Select the "Show all projects" checkbox, then select your project and from the drop-down list select the version of the engine for that particular project:

![Adding an asset to a project on Linux](/assets/img/posts/epic_games_marketplace_linux_add_asset.png)

Now you're all set! The asset pack is downloaded to the `Content/` folder of your UE4 project. 

<!-- links -->
[Lutris]: https://lutris.net/

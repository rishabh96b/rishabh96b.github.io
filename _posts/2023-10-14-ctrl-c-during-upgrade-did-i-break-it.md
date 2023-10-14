---
layout: single
title:  ""
date:   2023-10-14 20:30:00 +0530
tags: ubuntu linux apt troubleshooting
---

# `Ctrl+C` During Ubuntu Upgrade. Did I Break It?

## Introduction

Have you ever been in a situation where you decide to upgrade your operating system, only to realize it's it's not worth it and you start having second thoughts? Well, that's precisely what happened to me during my attempt to upgrade from Ubuntu 20.04 LTS to the newest, shiniest Ubuntu 22.04. Here's a tale of a `Ctrl+C` press, some troubleshooting, and a valuable lesson in keeping your system in tip-top shape.

## The Upgrade Decision

So, there I was, thinking it's time to upgrade my Ubuntu system to the latest LTS version. You know, keep things fresh and stay secure. But as the upgrade process started, I decided to check the performance benchmarks comparison of `20.04 LTS` and `22.04 LTS` which could be found here and realized that it is not worth upgrading to the next major LTS release.

## Ctrl+C: The Great Escape

In a fit of impatience, I hit Ctrl+C to cancel the upgrade midway. Seemed like a good idea (and the only idea) at the time – little did I know what was coming next.

## The Aftermath

After my Ctrl+C fiasco, Nothing major happened in my system. It could have been worse if I had terminated the upgrade a bit late. I figured I'd better revert to good old Ubuntu 20.04 LTS. To check the installation of packages using `apt`, it ran `sudo apt update`. But that's when things got interesting.

## The Mystery of somerville-kakuna

Instead of a smooth update, I got flooded with warning messages. Check this out:

```bash
...

W: Skipping acquire of configured file stable/source/Sources as repository 'https://download.docker.com/linux/ubuntu focal InRelease' does not seem to provide it (sources.list entry misspelt?)

W: Skipping acquire of configured file somerville-kakuna/binary-amd64/Packages as repository 'http://dell.archive.canonical.com jammy InRelease doesn't have the component somerville-kakuna (component misspelt in sources.list?)
.
.
W: Skipping acquire of configured file 'somerville-kakuna/cnf/Commands-amd64 as repository http://dell.archive.canonical.com jammy In Release doesn't have the component 'somerville-kakuna' (component misspelt in sources.list
```

Now that was quite a mouthful, right? It seems like I was stuck in a loop of 'somerville-kakuna' madness. The strange thing was that there was no mention of 'somerville-kakuna' in my` sources.list` file.

Stay tuned because this is where the troubleshooting adventure truly began.

## Troubleshooting Time
I checked `/etc/apt/sources.list` and saw the entries pointing to `jammy` repositories. So, I edited my `/etc/apt/sources.list` file and removed all entries related to `jammy` repos. As the repos for `focal` were commented, I uncommented them and DONE!

Begone Invalid Entries: Found the troublesome entry in my `sources.list` and removed it. Cleaned up my configuration file, making sure it was squeaky clean. Use `vim` or `nano` to make your changes.

```bash
sudo vim /etc/apt/sources.list
```

Ran update again but BUMMER! The same warnings were popping up. This compelled me to look into the `/etc/apt/` directory to dig about these warnings related to somerville. And I found these files specifically for somerville-kakuna and deleted those.

The Extra Repositories: Those sneaky additional repository configuration files in `/etc/apt/sources.list.d/` were the culprits. There it was, the unexpected entry causing the error.

The Ghostly Custom Repositories: Don't forget about those third-party repositories or PPAs you added ages ago. Some of them might be relics, so I gave them a thorough review.


Cleanup, Round Two: Remembered to delete the `sources.list.distUpgrade` file, which usually pops up during a distribution upgrade. If you've successfully upgraded, you can safely get rid of it.

Run apt update Again: After sorting out these issues, I ran `sudo apt-get update` to make sure my package management was back in business.

But there's more! Sometimes, your package manager can still act up, or a package might go rogue. In that case, here are a couple of trusty commands to keep in your back pocket:

```bash
sudo dpkg --configure -a
```
This command can help when something seems wonky with your package manager.

```bash
sudo apt --fix-broken install
```
If you've got a broken package causing trouble, this command might be your knight in shining armor.

## Conclusion

After this journey through the troubleshooting maze, success was finally mine. My system was back to the Ubuntu 20.04 LTS, performing like a champ. This whole escapade taught me a couple of vital lessons – don't be hasty with upgrades, be prepared for the unexpected, and there's always a solution to get your system back on track.

In the end, I decided that sticking with Ubuntu 20.04 LTS was the right choice for me. The performance was more consistent, and I didn't have to deal with newfangled quirks. But remember, the best decisions are made with a touch of patience and a dash of troubleshooting magic.

Stay curious and keep your system in top form!
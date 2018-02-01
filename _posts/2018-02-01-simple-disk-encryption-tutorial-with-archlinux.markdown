---
layout: post
title:  "Simple disk encryption tutorial with archlinux"
excerpt: ""
date:   2018-02-01 08:00:00 +0100
categories: archlinux
tags: archlinux tutorial
---

We all love [archlinux](https://www.archlinux.org/), or if we don't, we're using Fedora or Debian, and trolling is (almost) out of the scope of this article.

But let's be honest, even if the [wiki](http://wiki.archlinux.org/) is great, it can be intimidating sometimes. That's what happened to me yesterday. Here at [AlloMedia](http://www.allo-media.net), for security reasons, we're encrypting every laptop disk by default. As I'me using archlinux, I went to the wiki to follow how to "just" encrypt my disk. And well, [the page](https://wiki.archlinux.org/index.php/Disk_encryption) is a little bit overcharged, at the very least.

You have first to read about 10 pages of documentation, to learn that you now have to choose between 6 methods (*Loop-AES, dm-crypt +/- LUKS, Truecrypt, eCryptfs, EncFS*) and read every \*#! page to understand which one you may want to choose. I've choosen for you.

## Lvm on Luks

This is shipped with the kernel and seems to be the "default" on other distributions. It totally fits my needs: encrypt the whole system, swap included, and decrypt the system on boot using a passphrase.

If that's what you want to do too, follow the white rabbit, Neo.

## Following the rabbit

We will assume that you can erase your disk and start with a fresh install, if it's not the case, this article may not be for you. For the sake of this article, we will use `/dev/nvme0n1` as the main disk of the laptop. You may have something different like `/dev/sda`, that's fine, just replace `/dev/nvme0n1` by `/dev/sda` in the rest of the article.

First, follow the Archlinux installation guide to the point where they are asking you to @TODO.

### Safely erase your disk

Every page on the archlinux wiki says you should first be sure that no previous data will still be readable on your disk (if you have a new computer with nothing on it, this doesn't apply to you).

So we will start by putting random stuff on our disk to be sure to overwrite everything that may still be on it.

    command to erase



### Partitionning

@TODO

You should now have only 2 partitions, one for `/boot` that will not be encrypted, and another one that you will first encrypt, and then put your volumes on it (`/` and `swap`). In my case, the first partition that will be used for `/boot` is named `/dev/nvme0n1p1`, and the other one `/dev/nvme0n1p2`. You may have something like `/dev/sda1` and `/dev/sda2` if you partition naming scheme is not the same than mine.


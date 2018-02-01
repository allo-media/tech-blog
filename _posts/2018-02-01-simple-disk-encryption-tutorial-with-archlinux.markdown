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

This is shipped with the kernel and seems to be the "default" on other distributions. It totally fits my need: encrypt the whole system, swap included, and decrypt the system on boot using a passphrase.

If that's what you want to do too, follow the rabbit Neo.

## Following the rabbit


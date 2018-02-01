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

First, follow the [Archlinux installation guide](https://wiki.archlinux.org/index.php/Installation_guide) to the point just before __Format the partitions__, where they are telling you to modify the partition tables using __fdisk__ or __parted__. Here, you will need to erase all your partitions and create what's needed for the encryption.

### Clean and safely erase your disk

First, use `fdisk` or `gdisk` (if you're using UEFI) to wipe out what's on your disk, i.e. removing all existing partitions (of cours, this will delete all the data on your disk…).

For example, for `gdisk`:

    gdisk /dev/nvme0n1

    GPT fdisk (gdisk) version 1.0.3

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    Found valid GPT with protective MBR; using GPT.

    Command (? for help):

Use `p` to print your partition schema, and `d` to delete partitions. Once it's done, use `w` to write your changes to the disk (that is to say, __again__, deleting all the data on your disk) and quit `gdisk`.

Every page on the archlinux wiki says you should first be sure that no previous data will still be readable on your disk (if you have a new computer with nothing on it, this doesn't apply to you).

So we will put random stuff on our disk to be sure to overwrite everything that may still be on it. You can read the [wiki page](https://wiki.archlinux.org/index.php/Securely_wipe_disk#Random_data) or just run the following command:

    dd if=/dev/urandom > /dev/nvme0n1


### Partitionning

We now have a clean disk, let's create what's needed for our encrypted system, that is to say 2 partitions: a partition for `/boot` (that will not be encrypted) and another one for our encrypted volumes (where we will later put `/` and our `swap`).

Here is what we want to have (output of my `gdisk` with the `p` command):

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            2048         1050623   512.0 MiB   EF00  EFI System
       2         1050624      1000215182   476.4 GiB   8E00  Linux LVM

First, create the `/boot` partition of type `8300` (512Mo is a good size) following the [archlinux wiki](https://wiki.archlinux.org/index.php/EFI_System_Partition#Create_the_partition). I'm assuming you're using a system compatible with UEFI, if it's not the case, you may want to document yourself a little bit more using the wiki. Format the partition using _FAT32_.

Create the other partition of code `8E00` using the remaining space.

You should now have only 2 partitions, one for `/boot` that will not be encrypted, and another one that you will first encrypt, and then put your volumes on it (`/` and `swap`). In my case, the first partition that will be used for `/boot` is named `/dev/nvme0n1p1`, and the other one `/dev/nvme0n1p2`. You may have something like `/dev/sda1` and `/dev/sda2` if you partition naming scheme is not the same than mine.

You can then follow the (LVM on LUKS section)[https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS] section.

In short, here are the commands you should be running:

@TODO



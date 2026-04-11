---
title: Introduction to Yocto
date: 2025-05-01
---

Yocto is a collection of tools that help you create a custom Linux image from scratch.
It is not a Linux distribution itself, but rather a framework for creating custom Linux distributions.

Let's say you have your own version of a Single Board Computer like a Raspberry Pi that you plan to use as a room air quality monitor. You would want to have a minimal Linux image that boots up quickly and runs your air quality monitoring application. And so you would not require a lot of the packages that come with a standard Linux distribution like Fedora. That includes support for wireless adaptors, graphics drivers and so on.

## A brief intro

![The poky reference distribution](https://ik.imagekit.io/5jrct2yttdr/quartz/Introduction%20to%20Yocto/Drawing%202026-03-14%2002.21.26.excalidraw_5v1iYCrxJ.png)

- Poky is the reference distribution of Yocto.
- Bitkbake is the task executor and scheduler (Think of building Yocto as a set of building blocks, and bitbake is the tool that assembles them in the correct order.)
- Metadata is the task definitions

I'm not going to cover the initial setup. It's rather easy to follow their quick start guide - https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html

## Exploring files inside the Poky directory

First, go ahead and look for the latest LTS release of Yocto here.
In my case it's `scarthgap`.

**Exercise:** Instead of me giving you the command, try to clone the `poky` repository from `git://git.yoctoproject.org/poky` yourself. 
Make sure to checkout into your own local branch from `remote/refs/scarthgap/`. (Hint: check out `git clone --branch`)

Everything that starts with a `meta-*` is a layer. A layer is a collection of related recipes and configurations. 

```bash {15-19}
(base) amalr@l5pro:~/workspace/yocto/1/poky$ ls -la
total 104
drwxrwxr-x 12 amalr amalr  4096 Apr 12 01:53 .
drwxrwxr-x  3 amalr amalr  4096 Apr 12 01:52 ..
drwxrwxr-x  6 amalr amalr  4096 Apr 12 01:53 bitbake
drwxrwxr-x  4 amalr amalr  4096 Apr 12 01:53 contrib
drwxrwxr-x 21 amalr amalr  4096 Apr 12 01:53 documentation
drwxrwxr-x  8 amalr amalr  4096 Apr 12 01:53 .git
-rw-rw-r--  1 amalr amalr   769 Apr 12 01:52 .gitignore
-rw-rw-r--  1 amalr amalr   834 Apr 12 01:53 LICENSE
-rw-rw-r--  1 amalr amalr 15394 Apr 12 01:53 LICENSE.GPL-2.0-only
-rw-rw-r--  1 amalr amalr  1286 Apr 12 01:53 LICENSE.MIT
-rw-rw-r--  1 amalr amalr  2186 Apr 12 01:53 MAINTAINERS.md
-rw-rw-r--  1 amalr amalr   244 Apr 12 01:53 MEMORIAM
drwxrwxr-x 21 amalr amalr  4096 Apr 12 01:53 meta
drwxrwxr-x  5 amalr amalr  4096 Apr 12 01:53 meta-poky
drwxrwxr-x 10 amalr amalr  4096 Apr 12 01:53 meta-selftest
drwxrwxr-x  7 amalr amalr  4096 Apr 12 01:53 meta-skeleton
drwxrwxr-x  8 amalr amalr  4096 Apr 12 01:53 meta-yocto-bsp
-rwxrwxr-x  1 amalr amalr  1488 Apr 12 01:53 oe-init-build-env
lrwxrwxrwx  1 amalr amalr    33 Apr 12 01:53 README.hardware.md -> meta-yocto-bsp/README.hardware.md
lrwxrwxrwx  1 amalr amalr    14 Apr 12 01:53 README.md -> README.poky.md
-rw-rw-r--  1 amalr amalr  1034 Apr 12 01:53 README.OE-Core.md
lrwxrwxrwx  1 amalr amalr    24 Apr 12 01:53 README.poky.md -> meta-poky/README.poky.md
-rw-rw-r--  1 amalr amalr   529 Apr 12 01:53 README.qemu.md
drwxrwxr-x 11 amalr amalr  4096 Apr 12 01:53 scripts
-rw-rw-r--  1 amalr amalr  1064 Apr 12 01:53 SECURITY.md
-rw-rw-r--  1 amalr amalr    83 Apr 12 01:53 .templateconf
```

**Exercise:** There is a script in the list above that you need to `source` to initialize your build environment. Can you find it? Once you run it, it will create a `build` directory for you.

Inside the `build/conf/` directory, you will find the 'configuration' bit from the diagram above.

```bash {5,8}
(base) amalr@l5pro:~/workspace/yocto/1/poky$ ls -la build/conf/
total 40
drwxrwxr-x 2 amalr amalr  4096 Apr 12 01:59 .
drwxrwxr-x 3 amalr amalr  4096 Apr 12 01:59 ..
-rw-rw-r-- 1 amalr amalr   337 Apr 12 01:59 bblayers.conf
-rw-rw-r-- 1 amalr amalr   516 Apr 12 01:59 conf-notes.txt
-rw-rw-r-- 1 amalr amalr    77 Apr 12 01:59 conf-summary.txt
-rw-rw-r-- 1 amalr amalr 12576 Apr 12 01:59 local.conf
-rw-rw-r-- 1 amalr amalr    33 Apr 12 01:59 templateconf.cfg
```

This is where you would define your target machine, add layers, and configure build options. 

**Quick Task:** Open `local.conf` and see if you can find where the `MACHINE` variable is set. What is the default machine?

## What are layers made of?

Layers are made of recipe groups. \
And recipe groups are made of recipes. \
Recipes contain the instructions to build a package.

A package can be a library, an application, or a kernel module. Example: `nano`

![Recipe Structure](https://ik.imagekit.io/5jrct2yttdr/quartz/Introduction%20to%20Yocto/Drawing%202026-03-14%2002.21.26.excalidraw%201_4AbZjXVj4.png)

You can browse a list of pre-written layers and recipes on https://layers.openembedded.org/layerindex/branch/master/layers/

**Discovery Exercise:** Go to the layer index and search for `htop`. Which layer is it part of?

In most cases you'll find the one you need on this index. If not, you can create your own custom recipe.
And I'll be covering that in the next article.


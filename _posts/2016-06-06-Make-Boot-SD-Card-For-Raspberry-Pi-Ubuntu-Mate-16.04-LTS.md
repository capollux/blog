---
layout:   post
title:    "Make Boot SD Card For Raspberry Pi, Ubuntu Mate 16.04 LTS"
date:     2016-06-06 01:23:35 +0900
tags:     ubuntu raspberrypi
author:   junwoo
comments: true
---

```console
$ sudo apt-get install gddrescue xz-utils
$ unxz ubuntu-mate-16.04-desktop-armhf-raspberry-pi.img.xz
$ sudo ddrescue -D --force ubuntu-mate-16.04-desktop-armhf-raspberry-pi.img /dev/sdx
```

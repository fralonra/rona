---
title: "Repairing a corrupted file system"
date: 2020-02-05T21:56:20+08:00
author: "Fralonra"
cover: ""
tags: ["linux"]
keywords: ["linux", "file system"]
description: "Repairing a corrupted file system."
showFullContent: false
---

I occasionally ran into file system problems these days.
At the fisrt few times, I just ran `fsck` and then reboot, but it failed today when my system tried to mount my `home` device, and `systemctl status home.mount` gave me the following errors:
```
home.mount: Mounting timed out. Terminating.
home.mount: Mount process exited, code=killed, status=15/TERM
home.mount: Failed with result 'timeout'.
```
I noticed that I hadn't used [persistent block device naming](https://wiki.archlinux.org/index.php/Persistent_block_device_naming) in my `fstab`, so I edited the file to change the naming by label.
However, as a newbie, I forgot to set these labels before reboot, and as a result, my root partition left me as well! I found myself in the emergency shell (not in emergency mode anymore) and saw the terrifying words on my screen:
```
Bailing out, you are on your own. Good luck.
```
I then relied on an [antiX](https://antixlinux.com/) live-USB, and tried to mount the device to edit the `fstab`, but:
```
NTFS signature is missing.
Failed to mount '/dev/xxx': Invalid argument
The device '/dev/xxx' doesn't seem to have a valid NTFS.
Maybe the wrong device is used? Or the whole disk instead of a
partition (e.g. /dev/sda, not /dev/sda1)? Or the other way around?
```
It seemed like the device was treated as ntfs, though it was indeed an ext4 partition, so I added `-t ext4` to the command, but also with no luck that time:
```
mount: wrong fs type, bad option, bad superblock on /dev/xxx
```
I tried `fsck` and it gave me something like the following:
```
fsck: Attempt to read block from filesystem resulted in short read while trying to open /dev/xxx
Could this be a zero-length partition?
```
As the error suggested, the [superblock](https://ext4.wiki.kernel.org/index.php/Ext4_Disk_Layout#The_Super_Block) of the partition might have been corrupted.
I then found all the superblock backups by `mke2fs -n /dev/xxx`, and restored the superblock using one of them by `e2fsck /dev/xxx -b [superblock]`.
Finally, I ran `fsck` and `mount` again, and things went fine.

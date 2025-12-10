---
weight: 1
title: "Ch 1: A Linux Box"
bookToc: true
type: "docs"
bookFlatSection: true
---

# Ch 1: A Linux Box
This machine will be the core of my homelab.

## Hardware

<img src="/linux-box/dell_desktop.jpg" style="max-width: 400px"/>

I have an old Dell Optiplex 7010 Desktop laying around that I had purchased off ebay back in 2022 for around $100.

The sticker on top suggests it had been retired from a tech college. ♻️

Specs:
- Year: 2014
- CPU: i5-3470 3.20GHz 
- RAM: 8GB
- HD: 500GB 

## OS

There are a ton of Linux distros out there.

[Ubuntu Server](https://ubuntu.com/download/server) was my first thought, being that Ubuntu (desktop) is one of the most popular beginner distros.

However, upon further research, I decide on [Debian](https://www.debian.org/).

Reasons:
- comparatively minimalist
- known for stability and long uptime
- its the base for many other distros, including Ubuntu

## Installation

### Prepping bootable USB

I download the [Debian network installer ISO file](https://www.debian.org/CD/netinst/) to my laptop, which is another refurbished ebay purchase: a Thinkpad X1 Carbon Gen 9 running [Omarchy Linux](https://omarchy.org/).

Using the `lsblk` command, I discover the name of my attached usb drive (sda). 

![find usb dev](/linux-box/find_usb.png)

I then run `sudo mkfs -t ext4 /dev/sda` to reformat my drive with an ext4 file system.

Lastly, I copy the debian ISO onto my drive with `sudo dd if=./debian.iso of=/dev/sda status="progress"`.

- `if` is the input file (the ISO)
- `of` is the output file (my usb)
- `progress` option outputs progress to my terminal


### Running the installer

After attaching the USB to the Dell along with a spare monitor and keyboard, I turn on the power and hit f12 to enter the boot menu.

![boot menu](/linux-box/boot_menu.jpg)

From here I select the USB drive as the boot device and enter the Debian installer.

The only package I need in addition to the standard system utils is the SSH server (more on this in Ch 2)!

![pkg install](/linux-box/pkg_install.jpg)

<img src="/linux-box/debian_login.jpg" alt="debian login screen" style="max-width: 300px" />

Voila. A Linux box.

---
layout: post-no-feature
title: Automated ArchLinuxARM Install Guide
description: "Installing Linux from tar archives is cumbersome. Then also OSX does not support EXT filesystems. Two quick, simple and automated ways to fix the issue."
category: articles
tags: [raspberry, odroid, docker, ansible, tutorial]
---

TL;DR Either grab my docker [image](//hub.docker.com/peelsky/rpi-sdcard-builder) (recommended) or [provision](//github.com/peel/rpi-sdcard-builder) w/ vagrant

While building a wireless Raspberry Pi / ODROID-C2 cluster to run Docker containers [fn:1] I got myself into an awkward, blind mexican standoff with OSX and EXT4.
You know, the OSX stares at the EXT4, the EXT4 stares at the OSX, they both can't see nothing but it feels like things might get gory again.
I mean, looking at all that mess it might end up with me running yet another development VM, doing stuff, possibly producing a bunch of bash scripts, flashing SD cards and wiping the whole story from my concious self. I'd much rather prefer having a reproducible, cross-system solution to check out and run every now and then when adding new nodes to cluster, wiping the old ones, clean-installing the distro.
So here's how to conveniently get the precious ArchLinuxARM/Alpine/... `tar.gz`s onto those SD cards.

# Method 1: Vagrant

The first thing I came up with was obviously running all the stuff in a VM. 
For that Vagrant is an invaluable solution. The VM is provisioned with ansible. The
The original goal was to simply put an SD card, and have it running my devices in a few minutes.
As I had a few more SD cards to flash I needed a copy of the image file. So here it goes.

## HowTo

<script src="https://gist.github.com/peel/1067f6545a322916af5f.js"></script>

## Explained

The first issue I stumbled upon was the way VirtualBox handles (or not handles) Macbook's SD card reader.
In order to do so you need to create a rawdisk that mirrors a physical device. With VirtualBox this means issuing following command: `VBoxManage internalcommands createrawvmdk -filename sd_card.vmdk -rawdisk /dev/disk2`. This will create a vmdk image mirroring physical disk2. However to do so you need to unmount all the partitions from disk2 by running: `diskutil unmountDisk /dev/disk2` and setting looser permissions to disk2 with `sudo chmod 0777 /dev/disk2`. Then the `VBoxManage storageattach --storagectl SATAController --port 1 --device 0 --type hdd --medium sd_card.vmdk` command will mount the rawimage into the running VM. Oh, and the OSX will mount the disk automatically into your devices and locks VirtualBox from fiddling with disk geometry. So you'd need to unmount all the partitions again. Thankfully you can work with the `diskarbitrationd` daemon that monitors connected disks and automatically mounts them. However running `launchctl unload com.apple.diskarbitrationd` might not be the best idea as it results with a failure whenever trying to bring it back. However the service responds correctly to standard kill signals, so in order to stop it we'd send SIGSTOP signal and SIGCONT to continue. So after getting the service's PID with `sudo launchctl list | grep diskarbitrationd | awk '{print $1}'`, we'd issue kill ie. `sudo kill -SIGSTOP 71` and bring the service back with `sudo kill -SIGCONT 71`. And in between that we'd run provisioning of the VM. As you've most likely noticed in the previous section, 

### That's not how it really works. 

Vagrantfile is pretty much a ruby file that allows you to execute commands at given cycles of VM's life. Therfore all the cumbersome tasks have been codified in the file. First, I'm using GetoptLong to provide command flags for running the provisioning with. The VM will fail to provision if it's not configured properly. With the disk id set all the pre-tasks described above are ran along with the creation of a disk image, service status manipulation and attaching the disk image to the VM. The [provisioning itself]() is fairly simple and mirror's the process described at ArchLinuxARM's [installation guide]().


# Method 2: Docker (recommended)

Docker, no matter what you think about it, is primarily made for application containers. 
So it's better suited for exposing your applications rather than generating .img files, however, being able to do so and have the intermediary steps cached for future reference and simply download the container to generate the file is damn compelling. Which is probably why there are so many obvious misuses of Docker.
Anyways, here's how to get it working.

## HowTo

<script src="https://gist.github.com/peel/be19e1165a9856e2ce1f.js"></script>

Or... if you'd like to use another tar archive ie. perform the procedure for ODROID-C2:

<script src="https://gist.github.com/peel/fd0b424fdc0b99a07859.js"></script>


## Explained
That's all? Really? Well, yeah. The thing is the approach uses loop interfaces to create a 'virtual' disk device backed by an .img file that then gets shared with the local device. 
Please remember that the container is ran through Docker Machine which in case of any issues is capable to run the container.
All that the container does is pretty much downloading a raw archlinux image, necessary packages and a linux archive. All the rest happens through the Makefile which means with first steps done manually (tar download and packages installation) you can use the Makefile on a Linux box as well. Now that's insanely helpful use the Makefile on a Linux box as well. Now that's insanely helpful.
The Makefile itself is rather straight-forward it creates a backing img file with `dd if=/dev/zero of=sdcard.img bs=1M count=1850` and sets a loop device with `losetup ${ID} sdcard.img`, then partitions the image using `parted` into two partitions - boot for MBR and root with EXT4, untars onto the image and unmounts the image.

# Footnotes

[fn:1] Invalid forward reference

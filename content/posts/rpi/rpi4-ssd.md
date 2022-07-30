+++ 
date = 2022-07-30T09:00:11+02:00
title = "Migrate Raspberry Pi SD card to SSD"
slug = "" 
tags = "raspberry pi, storage"
categories = []
thumbnail = "/img/rpi-logo.png"
description = "Step by step migration instructions"
+++

Disk intensive applications such as kubernetes can run on Raspberry Pi 4, but you need to migrate from the SD card to an SSD drive.

Raspberry Pi can boot from USB, so you don't have to wear out your SD card anymore.



{{% portfolio image="/img/rpi4-ssd.jpeg" alt="Hardware" %}}

This is what you need to configure your raspberry pi:

 * A USB3 SATA or M.2 case
 * SSD disk
 * Os already installed on the old SD card

{{% /portfolio %}}



## Set up

Boot up your rpi4 on the SD card. Log in and connect the USB drive and check out your device names:

```
pi@raspberrypi:~ $ lsblk 
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0 111.8G  0 disk 
|-sda1        8:1    0   256M  0 part 
`-sda2        8:2    0  29.6G  0 part 
mmcblk0     179:0    0  29.8G  0 disk 
|-mmcblk0p1 179:1    0   256M  0 part /boot
`-mmcblk0p2 179:2    0  29.6G  0 part /
pi@raspberrypi:~ $
```
 
In this case the USB drive name is ```sda``` and ```mmcblk0``` is the SD card. If you're unsure, unplug the USB drive and plug it in again so you don't accidentially overwrite your SD card.

Next steps will erase all data on your SSD (the "sda" device) and copy your SD card image in one step. You've been warned.

```sh
pi@raspberrypi:~ $ sudo dd if=/dev/mmcblk0 of=/dev/sda status=progress
```

Grow the filesystem

```sh
pi@raspberrypi:~ $ sudo parted /dev/sda print
Model:   (scsi)
Disk /dev/sda: 120GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      4194kB  273MB   268MB   primary  fat32        lba
 2      273MB   40.0GB  39.7GB  primary  ext4
pi@raspberrypi:~ $ NEWDISKSIZE=$(sudo parted /dev/sda print | grep ^'Disk /dev/sda' | awk '{print $NF}')
pi@raspberrypi:~ $ sudo parted /dev/sda resizepart 2 ${NEWDISKSIZE}             # Expand filesystem
pi@raspberrypi:~ $ sudo fsck.ext4 /dev/sda2 
pi@raspberrypi:~ $ sudo tune2fs /dev/sda2 -L rootfs-SSD                         # Set a new label on the root fs
pi@raspberrypi:~ $ OLDUUID=$(awk '$2 == "/" {print $1}' /etc/fstab)
pi@raspberrypi:~ $ sudo mount /dev/sda2 /mnt/
pi@raspberrypi:~ $ sudo sed -i "s/${OLDUUID}/LABEL=rootfs-SSD/g" /mnt/etc/fstab # Mount the label as root in the new fstab
```

Now configure the machine to prioritize USB boot first

```sh
tom@k8s:~ $ sudo raspi-config
```

Advanced Options > Boot Order > USB Boot

{{% portfolio image="/img/rpi4-stp1.png" alt="Step1" %}}{{% /portfolio %}}
{{% portfolio image="/img/rpi4-stp2.png" alt="Step2" %}}{{% /portfolio %}}
{{% portfolio image="/img/rpi4-stp3.png" alt="Step3" %}}{{% /portfolio %}}
{{% portfolio image="/img/rpi4-stp4.png" alt="Step4" %}}{{% /portfolio %}}

Shut down your raspberry and take out the SD card before powering it on again.


{{% portfolio image="/img/rpi4-stp1.png" alt="Step1" %}}{{% /portfolio %}}
{{% portfolio image="/img/rpi4-stp5.png" alt="Step5" %}}{{% /portfolio %}}



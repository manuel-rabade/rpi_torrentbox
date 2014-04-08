Raspberry Pi Torrentbox
=======================

This is a Raspberry Pi Torrentbox and NAS server example.

The Raspberry Pi is a model B 512 MB that runs ArchLinux ARM
2014.03 with a 2 TB external USB hard drive.

Setup
-----

Steps to reproduce this example:

### Install 

Write ArchLinux ARM on your Raspberry Pi SD card and boot the system.

Update the system:

    # pacman -Syu

### Networking

Set hostname: 

    # hostnamectl set-hostname torrentbox

Modify `/etc/netctl/eth0`

Enable netctl profile:

    # systemctl disable netctl-ifplugd@eth0.service
    # netctl enable eth0

Reboot to apply changes:
    
    # reboot

### Date & Time

Set timezone:

    # timedatectl set-timezone America/Mexico_City

Install and enable OpenNTPD:

    # systemctl stop ntpd
    # pacman -S community/openntpd
    # systemctl enable openntpd

Reboot to apply changes:

    # reboot

### External USB hard drive

Install parted:

    # pacman -S parted

Partition the hard disk:

```
# parted -a optimal /dev/sda
GNU Parted 3.1
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) unit mb
(parted) print                                                            
Model: ADATA NH03 (scsi)
Disk /dev/sda: 2000399MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start  End  Size  File system  Name  Flags

(parted) rm 1
(parted) mkpart primary linux-swap 1 2049
(parted) mkpart primary ext4 2050 2000399
(parted) print                                                            
Model: ADATA NH03 (scsi)
Disk /dev/sda: 2000399MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End        Size       File system     Name  Flags
 1      1.05MB  2049MB     2048MB     linux-swap(v1)
 2      2050MB  2000399MB  1998349MB

(parted) quit
```

Create filesystems:

    # mkswap /dev/sda1 
    # mkfs.ext4 /dev/sda2

Create mountpoint:

    # mkdir /usb

Modify `/etc/fstab`

Reboot to apply changes:

    # reboot

### BitTorrent

Install Deluge:

    # pacman -S deluge python2-mako

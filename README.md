Raspberry Pi Torrentbox
=======================

This is a Raspberry Pi Torrentbox and NAS server example.

The Raspberry Pi is a model B 512 MB that runs ArchLinux ARM
2014.03 with a 2 TB USB Hard Disk Drive.

Setup
-----

Steps to reproduce this example:

### Install 

Write ArchLinux ARM on your Raspberry Pi SD card and boot the system.

Update the system:

    # pacman -Syu

Change root password:

    # passwd

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

### External USB HDD

Install parted:

    # pacman -S parted

Partition USB HDD:

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

    # mkdir /mnt/usb

Modify `/etc/fstab`

Mount all filesystems in fstab:

    # mount -a

### BitTorrent

Install Deluge:

    # pacman -S deluge python2-mako

Create torrents directories:

    # mkdir /mnt/usb/torrents
    # mkdir /mnt/usb/torrents/{incoming,complete}
    # chown -R deluge:deluge /mnt/usb/torrents

Enable Deluge:

    # systemctl enable deluged
    # systemctl enable deluge-web

Start Deluge:

    # systemctl start deluge-web
    # systemctl start deluged

Configure Deluge:

1. In your browser go to http://192.168.1.10:8112
2. The default password is `deluge`
3. Connect to localhost:58846
4. Click `Preferences` button
5. In `Downloads` category change:
  * Download to: `/mnt/usb/torrents/incoming`
  * Move completed to: `/mnt/usb/torrents/complete`
6. In `Interface` category change the default password (don't forget to
   click `Change` to save it)
7. Click `Ok` to finish

### Windows Shares (optional)

Install SAMBA:

    # pacman -S samba

Create `/etc/samba/smb.conf`

Create group:

    # groupadd -g 1000 lan

Create user:

    # useradd -u 1000 -g lan -G deluge -s /bin/false lan

Add user to PDB:

    # pdbedit -a -u lan

Create shared directories:

    # mkdir /mnt/usb/{docs,files}
    # chown lan:users /mnt/usb/{docs,files}
    # chmod 775 /mnt/usb/{docs,files}

Enable SAMBA:

    # systemctl enable smbd
    # systemctl enable nmbd

Start SAMBA:

    # systemctl start smbd
    # systemctl start nmbd

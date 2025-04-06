---
title: Home Server
date: 2025-04-06
layout: post
categories: [post]
---

One of my first uses for RaspberryPi was a media player as we did not have a smart TV back then. I had Open Source Media Center (OSMC) running pretty well, connected to the TV which allowed me to play movies from a flash drive. Not long after, it became obvious that I need to have a torrent client as well so I can directly download movies on demand. I would also need a storage solution along with that to store my movie collection. It was then that I learned about Open Media Vault (OMV) as a possible NAS solution. Eventualy, I had one running providing the torrent client and as the media server for my OSMC box.


That was a decade ago. The OSMC box has since outlived its purpose and we would simply Netflix and chill nowadays. The OMV server, however, is still alive and well, and has undergone several hardware and software upgrades over the years. It has also taken on a new purpose which is the backup of my personal pictures from my phone.

Here is my recent setup, with plenty of pictures and descriptions to give you an idea of how it works and how you might replicate it in your home as well.
<br>
<br>

### Hardware

I am using OrangePi 3B which is cheaper compared to a similar-spec'd RaspberryPi. I got the 2GB RAM variant bundled with an external 32GB eMMC storage. This happens to be the most capable SBC I own so I allocated it to the home server. Board details here:
<http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_3B>

![Image 5](/assets/images/2025-04-06/h0.png)  

<br>
Here is the completed unit in a neat hard-shell package.  

![Image 5](/assets/images/2025-04-06/h1.png)  

<br>
Lower side showing the USB-C for power supply, ethernet jack, and output vents. The vents, together with the intake fan, are just enough to maintain positive pressure inside the unit. Although the OPi provides Wi-Fi capability, I still opted for wired networking. Power is provided by an official RaspberryPi USB-PD wall adapter.  

![Image 6](/assets/images/2025-04-06/h2.png)  

<br>
For the storage, the board boots from the 32GB eMMC. I initially used a USB flash drive as temporary download location for my torrent files, which proved to be a bad idea, as the drive became inaccesible just after a year. USB flash drives, I realized, are not optimized for such frequent writes. So I switched to a 128GB SD card marketed for CCTV applications. I opted for a card reader, keeping the on-board SD card socket free in case I needed to boot from it in emergency cases. Primary storage is provided by a 2TB HDD containing all my pictures and documents.  

![Image 7](/assets/images/2025-04-06/h3.png)  

![Image 8](/assets/images/2025-04-06/h4.png)  

<br>
Here is the unit powered up, hanging on the wall via a cheap zip tie! This is the first thing I'll grab in case of fire!  

![Image 10](/assets/images/2025-04-06/h8.png)  
<br>
<br>

### Software

Visit OpenMediaVault page here for details:  
<https://www.openmediavault.org/> 

![Image 1](/assets/images/2025-04-06/s1.png)  

<br>
This is my OMV home page, hosted on my local server. There are various home page customizations for status and monitoring.  
I won't go into the details of the individual setup, but the main configurations include setting up the networking, storage, local services (including SMB share for file access within the home network), and setting up the Docker services.  
Setup guide should be available in various sources in the web.     

![Image 2](/assets/images/2025-04-06/s2.png)  

<br>
One of the cool things about OMV is that it integrates docker functionality as a plugin. Adding a service is as simple as copying a sample Docker compose file, customizing the storage and network options, then running it!  

I have a qBittorrent service for my movie downloads, whenever needed.

![Image 3](/assets/images/2025-04-06/s3.png)  

<br>
I also have a NextCloud service to manage my personal files. Together with the accompanying Android app, it auto-uploads pictures from my phone, organized in year/month folders. I don't want to risk the security of my data, so I configured everything strictly for local access only.  

![Image 4](/assets/images/2025-04-06/s4.png)  
<br>
<br>

### Image Setup

I'll include my image setup steps here, primarily for my own use, should I need to reproduce the setup in the future.  

```
# Download image from OrangePi (Orangepi3b_1.0.6_debian_bookworm_server_linux5.10.160.7z)
# http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-3B.html
# Burn image using RPI imager

# ETH issue with latest OPi image... need to add following
$ sudo nano /etc/rc.local
#-----------------------
io -4 0xFDC60284 0X3f3f3f3f
io -4 0xFDC6028C 0x003f003f
io -4 0xFDC60298 0x3f00ef00
io -4 0xFDC6029C 0x3f3f3f3f
io -4 0xfdc60388 0xFFFF0049
#-----------------------

### Update date-time
$ timedatectl
$ sudo systemctl stop chrony
# should be OK to skip ----> $ sudo timedatectl set-time "2025-1-15 12:30:00"
$ sudo timedatectl set-timezone Asia/Manila

$ sudo nano /etc/chrony/chrony.conf
#-----------------------
pool time.nist.gov iburst
#-----------------------

$ sudo systemctl stop chrony
$ sudo systemctl start chrony

### Enable SSH
$ sudo systemctl stop ssh
$ sudo nano /etc/ssh/sshd_config
#-----------------------
Port 22
PermitRootLogin no
PublicKeyAuthentication yes
PasswordAuthentication yes
KbdInteractiveAuthentication no
UsePAM no
#-----------------------
> sudo reboot

# Can now swith to headless...
$ sudo apt-get update && sudo apt-get upgrade

### Copy image
# Use WinSCP gui to transfer image Orangepi3b_1.0.6_debian_bookworm_server_linux5.10.160.img to orangepi

# See available external flash
$ ls /dev/mmcblk*boot0 | cut -c1-12

# This should wipe the external flash
$ sudo dd bs=1M if=/dev/zero of=/dev/mmcblk0 count=1000 status=progress
$ sudo sync

# This will write the image to external flash...
$ sudo dd bs=1M if=Orangepi3b_1.0.6_debian_bookworm_server_linux5.10.160.img of=/dev/mmcblk0 status=progress

### Boot from external flash
# SD card can be removed at this point

# If wont boot, might need to clear the SPI flash. See OPI 3B manual for steps using flasher tool:
# (http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_3B)

# Username/ password: *****/ *****

### Initial setup
# Repeat the image setup above:
#   - Ethernet fix
#   - Update/upgrade
#   - Update date/time
#   - Configure SSH

##########
### Install OMV
# Source: https://github.com/OpenMediaVault-Plugin-Developers/installScript?tab=readme-ov-file

$ sudo wget https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install
$ sudo chmod +x install
$ sudo ./install -n
# Get coffee at this point...

$ sudo apt-get update && sudo apt-get upgrade
$ sudo omv-upgrade

### Configure OMV
## Good reference: https://forum.openmediavault.org/index.php?thread/49357-omv-quick-configuration-guide/

# Various common settings...
# Go to Network > General > Hostname: *****, Domain name: ***** 
# Go to System > Date & Time > Time servers: time.nist.gov

# Physically mount drives and confirm in Storage > Disks
# Go to Storage > File Systems and hit blue arrow to add existing disks
# Go to Storage > Shared Folders and add each disks
#    - Relative path --> /

# Go to Services > SMB/CIFS > Settings > Enable
# Go to Shares > Public > Guests allowed
# Confirm access in Windows Network files

## Install Docker Compose
# System > Plugins > openmediavault-compose

## Configure Docker Compose
# https://wiki.omv-extras.org/doku.php?id=omv7:omv7_plugins:docker_compose

## Add User
# Add appuser... group: users
# UID should be 1001. GID should be 100

### Compose files

### qbittorrent
-----------------------------------------
---
# https://hub.docker.com/r/linuxserver/qbittorrent
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1001
      - PGID=100
      - TZ=Asia/Manila
      - WEBUI_PORT=8080
    volumes:
      - /appdata/qbittorrent/config:/config
      - /srv/dev-disk-by-uuid-*****/downloads:/downloads
    ports:
      - 8080:8080
      - 6881:6881
      - 6881:6881/udp
    restart: unless-stopped

-----------------------------------------

### nextcloud
-----------------------------------------
---
# https://hub.docker.com/r/linuxserver/nextcloud
# https://docs.linuxserver.io/images/docker-nextcloud
# only x86-64 and arm64 are supported
services:
  nextcloud:
    image: lscr.io/linuxserver/nextcloud:latest
    container_name: nextcloud
    environment:
      - PUID=1001
      - PGID=100
      - TZ=Asia/Manila
    volumes:
      - /appdata/nextcloud/config:/config
      - /srv/dev-disk-by-uuid-*****/nextcloud:/data
    ports:
      - 8443:443
    restart: unless-stopped

-----------------------------------------

## Username/ password: *****/ *****

sudo nano /appdata/nextcloud/config/www/nextcloud/config/config.php
# add
'check_data_directory_permissions' => false,


```
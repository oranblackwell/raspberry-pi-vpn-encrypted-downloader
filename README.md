# RPi-Box

The Internet is full of free, legally downloadable movies and TV shows on many different websites. My favorites are [The Internet Archive](https://archive.org/details/feature_films), [Retrovision](http://retrovision.tv/) and the Sony owned [Crackle](https://www.crackle.com/about). The following is a guide on how to setup a fully automated, always-on, low-powered media download center using the new Raspberry Pi 4 Model B, or the older 3 Model B+ or 3 Model B.

The following guide will take you through all the steps needed to  take a fresh raspberry pi and setup all the tools required to have a VPN-encrypted download box running Docker Compose instances of Sonarr and Radarr (for Tv and Movies respectively), supporting downloads via torrents and usenet (setup via by DockStarter), topped off with real-time monitoring via RPi Monitor, also running as a Docker Compose instance.

[Overview](#individual-steps)

**Author:** Oran Blackwell

*Revised:* August 09, 2019

-----------------------------------------------------------------

## Overview

### Features 
 - **[Raspbian Buster Lite](https://www.raspberrypi.org/downloads/raspbian/)** - *Raspbian is the official operating system for all models of the Raspberry Pi. <small>(\*see [Note](#Note-on-the-Operating-System) below)</small>*
 - The following are provided via **[DockSTARTer(]https://dockstarter.com/) - The main goal of DockSTARTer is to make it quick and easy to get up and running with Docker.
   * **[Sonarr](https://github.com/Sonarr/Sonarr)** - *For automatically downloading, sorting and renaming TV shows via Usenet and/or BitTorrent. (Alternative to SickRage)*
   * **[Radarr](https://github.com/Radarr/Radarr)** - *For automatically downloading, sorting and renaming movies via Usenet and/or BitTorrent. Fork of Sonarr, but for movies (Alternative to CouchPotato)*
   * **[NZBGet](https://github.com/nzbget/nzbget)** - *A lightweight USENET download client, known for being fast and lightweight - ideal for RPi.*
   * **[Deluge](https://deluge-torrent.org/)** - *A lightweight BitTorrent client.*
   * **[Jackett](https://github.com/Jackett/Jackett)** - *An API to provide additional torrent trackers.*
 - **[RPi-Monitor](https://github.com/XavierBerger/RPi-Monitor)** - *To perform real time monitoring of the RPi - Uptime, CPU and Memory Usage, CPU Temperature, Disk Usage, optional Network monitoring.*
 - **Network-Attached Media Storage** - *My version connects to a 3TB WD My Cloud drive I picked up for cheap*
 - **USB-attached hard-drive** (*Optional*) - *Used to store incomplete downloads and perform load-intensive post-processing extractions before moving to final location on Network-Attached Media Storage (above). I've an old 500GB HDD*
 - **Modular Installation** - *Thanks to DockSTARTer each section of the installation process separate allowing you to leave out the sections that you mightn't need. For example, if you didn't want TV Shows you would omit the Sonarr stage or if you didn't want to use Torrents you would omit the Deluge and Jackett stages.*
 
 
#### Note on the Operating System
 - A ***headless*** Operating System mean that there is no GUI - Graphical User Interface - meaning that you are **only** able to interact via the command line and SSH. This results in a more light-weight system, ie. better overall system performance but at the cost of graphical and interaction features.
	  
### Individual Steps

1. Getting the Raspberry Pi ready to begin.
1. Setting up the network-drive.
1. Setting up the USB drive.
1. Installing DockSTARTer which takes care of Docker, Docker Compose Sonarr, Radarr, Deluge, NZBGet etc.
1. Installing OpenVPN.
1. Overcoming running OpenVPN alongside Docker.
1. Adding a Docker Compose instance of RPi-Monitor (for monitoring the resource usage of your RPi-Box) via DockSTARer's extensionss protocall.


#### My full setup 
#UPDATEME!

| Description 									|    		| Details      |
| :---         									|   :---: 	|          ---: |
| RPi version       							| 			| Raspberry Pi 3 B Plus    |
| RPi OS &nbsp;  &nbsp;  &nbsp;  &nbsp;  &nbsp;	| 			| Raspbian Lite Stretch (2018-04-18-raspbian-stretch-lite.img)    |
| Distribution 									|  			| Raspbian GNU/Linux 9 (stretch)      |
| Kernel version 								|  			| Linux 4.14.34-v7+ armv7l      |
| Firmware 										|  			| #1110      |
| SD Card 										| 			| Kingston 32Gb Micro SD Card - Class10      |
| Mounted via 									| 			| [Etcher](https://etcher.io/) (for Windows 10 x64)      |
| Network HDD 									| 			| 3TB WDMyCloud Home      |
| Second HDD (USB)								| 			| XXXXXXX      |


## Getting Started

### Getting the RPi up and running #

#### Flash the OS
The first step is to install a Raspberry Pi operating system onto an SD card.  I won't cover these steps here as there are thousands of online tutorials already written for this. Official documentation can be found [here](https://www.raspberrypi.org/documentation/installation/installing-images/). I *highly* recommend [Balena Etcher+](https://www.balena.io/etcher/) 

#### Enable SSH access

After creating the image on the SD card, load the card into your RPi, connect it to a monitor (temporarily) and boot it up. 


##### Default Login Credentials # 

 - **Username:** pi
 - **Password:** raspberry

If your installed a headless OS, you'll already be in the CLI (Command Line Interface), if you chose to have a GUI, you'll need to launch a terminal window.

Once you're in the CLI, run `sudo raspi-config` and change the following:
 - Change the default password by choosing 1 from the first list ("Change User Password").
 - Enable SSH by choosing 5 from the first list ("Interfacing Options")and P2 from the second ("SSH").
 - (Optional) Choose 2 from the first list ("Network Options")
   - Choose N1 to change the hostname to something more distinguishable on your network - I chose **RPi-Box**.
   - Choose N2 to setup a Wi-Fi connection.


When you've finished making your changes, choose "Finish" and reboot the RPi when prompted. Login with the same username from the previous step, but with the updated password.

Run `ifconfig`. You'll need to scan the returned text for the IP address for your chosen connection (wired or wireless). It such look something like "172.16.254.1". Generally, this will be on the second line of the relevent section:
 - For Wi-Fi connections look under the wlan0 section.
 - For wired connections look under eth0.

Take note of the IP address.

Alternatively, you can use `hostname -I` to display the IP addresses associated with your RPi.

#### Logging in remotely
(See enabling SSH above)
SSH is built into Linux distributions and Mac OS. For Windows and mobile devices, third-party SSH clients are available. See the following guides for using SSH with the OS on your computer or device:

[Linux & Mac OS](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md)
[Windows](https://www.raspberrypi.org/documentation/remote-access/ssh/windows.md)
[iOS](https://www.raspberrypi.org/documentation/remote-access/ssh/ios.md)
[Android](https://www.raspberrypi.org/documentation/remote-access/ssh/android.md)

### Updating the Operating System
The first thing you need to do is update the installed OS to the latest version. From a terminal (either directly on the Pi or via SSH) run the following:

```
sudo apt-get update
sudo apt-get dist-upgrade -y
```

Select "y" if prompted to allow changes.

Once everything is finished reboot your deivce via: `sudo reboot now`

# Setting up the hardrives
You can check if your drive is available  for mounting via:
`sudo blkid` or `lsblk`

Depending on the type of drive you are trying to mount, you willl need to install the appropriate set of utilities. You may need to run something like:

`sudo apt-get install cifs-utils`
or
`sudo apt-get install nfs-common`

For the filesystem on my 3TB WD My Cloud I needed to run `sudo apt-get install ntfs-3g`

Once the utilities are installed you will need to create an emtpy directory to which you will mount the drive. The convention is to mount these within your /mnt directory.

I choose to name my network mount "net" located here: /mnt/net 
Another approach is to mount it within your User's home direcory, ie. /usr/pi for our default installation. This is often shortened to "~" (pronounced tilde). So you can cd to your user's home directory by typing  ```cd /usr/pi``` and/or ```cd ~```

## Setting up the network attached storage

After installing the appropriate drivers (above)

I created my mount point via:
```
sudo mkdir /mnt/net
```

I then set the ownership of that folder to my current user and gave appropriate permissions.
```
sudo chown -R $USER /mnt/net
sudo chmod -R 775 /mnt/net
```

#### The next step will auto-mount this drive on every system boot. This section is TRICKY. Take your time.

When you have worked out all the variables, the entire process can by simplified to just a single line.

``` 
echo "//192.168.1.101/Media /mnt/net cifs defaults,rw,uid=pi,gid=pi,username=pi,password=XXXXXXXXXX,x-systemd.automount 0 0" | sudo tee -a /etc/fstab > /dev/null
```

What this does is echo (or print) the text inside the inverted commas into the file /etc/fstab. This file is read each time your system boots.

##### Please Note
Running this command multiple lines will add multiple lines to your /etc/fstab which may cause issues. You can maually edit this file and remove any previously entered lines.

#### Variables
```
ABS_DIR_ON_RPI="/mnt/net" 
  # Absolute path to mounted directory.
  #+ Format: slash before, no slash after. (eg /mnt/net)

Alternatively you could mount your drive within your user's home directory (~). in which case your direcories would be:

ABS_DIR_ON_RPI="~/wdMyCloud" 
  # Absolute path to mounted directory.
  #+ Format: slash before, no slash after. (eg /home/pi/wdMyCloud
 
I've given my drive a static IP via my router's admin page. 

IP_OF_NET_DRIVE="//192.168.1.101"   
  #  The IP address of the network attached drive.
  #+ Format: Two slashes ("//") before, no slash after. (eg //192.1.123.123)

MEDIA_DIR_ON_NET_DRIVE="/media"     
  # The directory to the media on the network drive/share.
  # So, your drive may already have directores on it, for example, documents, music etc. On my drive all my media is located with a sub-directory called "media"
  #+ Format  Slash before. (eg /path/to/media)
  
  Depending on your drive you may need to create a user which has the appropriate access to read and write to the drive. I needed to create a user called "pi" with the password "<mypassword>"

NET_DRIVE_USER="pi" 
NET_DRIVE_PASSWORD="<mypassword>"
  # If your network drive requires login or share credentials, minimum of Read and Write access required
  #+ Write Access required in order to create thumbnails & previews, add subtitles, posters etc. 
  #+ and also delete episodes after watching via setting in Plex.

sudo mount -t cifs -o guest ${IP_OF_NET_DRIVE}${MEDIA_DIR_ON_NET_DRIVE} ${ABS_DIR_ON_RPI}
  # This mounts the drive on THIS boot
  #+ NOTE: You may need to change cifs to your relevant drive, eg nfs etc.

echo "$IP_OF_NET_DRIVE$MEDIA_DIR_ON_NET_DRIVE $ABS_DIR_ON_RPI cifs defaults,rw,uid=1000,gid=1000,username=$NET_DRIVE_USER,password=$NET_DRIVE_PASSWORD,x-systemd.automount 0 0" | sudo tee -a /etc/fstab
  # This mounts the drive on FUTURE boots by appending a line to the end of "/etc/fstab".
  #+ Running this part multiple times will keep adding to the file, which could cause
  #+ future conflicts. You need to edit the file manually to remove.
  #+ NOTE: You may need to change cifs to your relevant drive, eg nfs etc.

```

For reference, here are two examples of a full line that's appended to /etc/fstab:
```
//192.168.1.101/Media /mnt/net cifs defaults,rw,uid=1000,gid=1000,username=pi,password=XXXXXXXXXXX,x-systemd.automount 0 0
```
```
//192.168.1.101/Media /home/pi/wdMyCloud cifs defaults,rw,uid=1000,gid=1000,username=pi,password=XXXXXXXXXXX,x-systemd.automount 0 0
```

You should now be able to `cd` into your network directory from the RPi-Box and view any pre-existing media. I strongly recommend your run 'sudo reboot' and double check the drive is still mounted afterwards. If its not you will definelty run into dificulties later on.

After rebooting type `cd /mnt/net` and hopefully you'll be in your network-attached hard-drive's location.


## Mounting a USB HDD (for temporary downloading and processing)

This is very much the same as the netowrd attached HDD aboce with only a few differences.

Instead of using an IP address you will use the ID assign to the dirve which you can find by running `lsblk`. It should be something along the lines of sda1 or sda2.

You then create the appropriate mount point, grant the correct permissions and add the correct line to your /etc/dfstab.

```
sudo mkdir /mnt/usb
sudo chown -R $USER /mnt/usb
sudo chmod -R 775 /mnt/usb

echo "/dev/sda1 /mnt/usb ntfs-3g defaults,rw,ofail,x-systemd.device-timeout=1  0 0" | sudo tee -a /etc/fstab > /dev/null
```


Dont forget to reboot and double check that both these mount points are auto-mounted.




```
# ########################################################################### #
# =========================================================================== #
```

```
# ########################################################################### #
# =========================================================================== #
```

```
echo -e "\e[1;35m
Jackett:     http://RPi_IP:9117
Deluge:      http://RPi_IP:8112
Sonarr:      http://RPi_IP:8989
Radarr:      http://RPi_IP:7878
NZBGet:      http://RPi_IP:6789
RPi Monitor: http://RPi_IP:8888

The default password for deluge is 
Password:   deluge

The default credentials for NZBGet are 
User:       nzbget 
Password:   tegbzn6789

For security reasons it is recommended to change the default credentials.

Give system some time to start services before trying to access above via browser from your host
===========================================================================================================\n${NC}"
```

*I used a great [post by GreenFrog](http://greenfrog.eu5.net/rpilds.php) as a starting point for this article.*

(Ctrl+Shift+M -  Open Notepad++ Markdown Split-screen Plugin)

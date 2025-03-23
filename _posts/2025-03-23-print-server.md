---
title: Print Server
date: 2025-03-23
layout: post
categories: [text]
---
Our first printer was an Epson L3110, its a 3-in-1 printer with scan, copy, print functionality, but with no Wi-Fi connectivity. At that time, I did'nt thought it is a necessity, since we had the printer right next to the PC afterall. Wrong assumption. It became increasingly needed as most of our print jobs originated from our phones.

After some reseach, I learned about print servers and that it can actually be implemented in a RasberryPi. I had a couple of old ones gathering dust somewhere, so I knew I can create one fairly easily. I had it working, following some online guides, and it finally allowed us to print directly from our phones withouht the need to turn on the PC.

Fast forward, I had to retire the Epson printer after several expensive repairs that I did (changed main board, print head cleaning and replacement 3x, etc...). This time I opted for a basic, single-function printer with no Wi-Fi, of course, as I already have my handy print server!
<br>

### Hardware Setup

This is our working printer, HP Ink Tank 115:
![Image 1](/assets/images/2025-03-23/print_server_01.jpg)  
<br>
Here is the print server unit. Left side cables connect to the printer: power jack and USB-A data cable. Right side is the AC cable:
![Image 3](/assets/images/2025-03-23/print_server_03.jpg)  
<br>
Inside the unit is an old RaspberryPi 1 B+. Wi-Fi connectivity is provided by a USB Wi-Fi dongle (not visible here). AC power goes to a 5V wall adapter (obviously not to be mounted on the wall) to provide 5V to the RPi. Same AC input branches to the other side to provide power to the printer, so no need for a separate power plug:
![Image 4](/assets/images/2025-03-23/print_server_04.jpg)  
<br>

### Software Setup

I got this from my setup notes:

```
## Setting up CUPS (non-docker) print server in RPI 1 B+ with Ubuntu Server
$ sudo apt update
$ sudo apt install cups
$ sudo usermod -a -G lpadmin pi
$ sudo cupsctl --remote-any

## Install HP ink tank 115 driver 
$ sudo apt install printer-driver-hpcups hplip cups-pdf

## Optional test page script. Used to auto print test page upon powerup 
$ sudo lpadmin -d HP115
$ lpstat -v
$ PRINTER=HP115
$ export PRINTER

$ crontab -e

# Add to crontab
@reboot /home/printer/myscript.sh

$ sudo chmod +x /home/printer/myscript.sh

#!/bin/bash
sleep 5
lp -d HP115 -o fit-to-page /usr/share/cups/data/testprint

```
<br>
### Configure and Test
CUPS provide an admin webpage where everything can be setup, including printer discovery/ driver selection, printer preferences, printjob monitoring, and a lot more:
![Image 5](/assets/images/2025-03-23/print_server_05.jpg)  

![Image 6](/assets/images/2025-03-23/print_server_06.jpg)  
<br>
The comforting sight of the CUPS printer test page!
![Image 7](/assets/images/2025-03-23/print_server_07.jpg)  
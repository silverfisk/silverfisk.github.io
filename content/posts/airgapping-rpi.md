+++ 
date = 2022-05-05T22:16:17+02:00
title = "Airgap your Raspberry Pi on a shoestring budget"
slug = "" 
tags = "security"
categories = []
thumbnail = "/img/rpi-logo.png"
description = "Airgap install a Raspberry Pi with a cheap ESP-01 Programmer"
+++

Airgapping is a method used when you want to be reasonable sure your device wasn't tampered with.
Regardless if you want to install a [Seedsigner](https://github.com/SeedSigner/seedsigner/blob/dev/docs/manual_installation.md) or something else. The best is to never have had it connected to your network in the first place.

Here's how I used a $1 USB serial programmer for an ESP-01 to configure an off-line Raspberry Pi. I'll also show how to transfer files to your machine with this oldschool method.



{{% portfolio image="/img/esp01-usb-programmer.jpg" alt="ESP01 Programmer" %}}

Most attackers (unless "stuxnet-level") use the IP protocol to interfer with their victims.
Another vector is to interfer with the USB cables used to communicate with a device.

With this method, no TCP/IP nor USB devices was ever connected. This makes verification of all steps much easier.

Furthermore, the computer draws power from the GPIO pin so there is no need to power it from the USB port.

{{% /portfolio %}}

{{% portfolio image="/img/esp01-usb-serialcables.jpeg" alt="Programmer" %}}

This is what you need to configure your raspberry pi:

 * A USB Serial programmer (I got mine for a dollar from AliExpress)
 * Four Jumper wires
 * A linux computer with the *screen* tool installed

{{% /portfolio %}}


## Set up

Connect a jumper wire in each one of the four pins (TX, GND, 3V+ % RX).
{{% portfolio image="/img/ESP01-pins.png" alt="Programmer Pins" %}}

{{% /portfolio %}}


Each one should go to the GPIO header with the same marking on the Raspberry PI:
{{% portfolio image="/img/connections-rpi.jpg" alt="Connections" %}}
{{% /portfolio %}}


### Preparation of the Raspberry Pi

Flash the Raspberry Pi image onto the SD card. Re-mount the SD card and edit ```config.txt```. The last line should say:
```
enable_uart=1
```

* Put the SD card in the RPi
* Connect the USB serial interface to your computer. It will power up your RPi directly.


## Connect the console

Open a terminal on your computer

```sh
sudo usermod -a -G dialout $(whoami)    # Give yourself access to the serial device
newgrp
screen /dev/ttyUSB0 115200
```

## Copy files over serial connection

All files you want to copy have to be base64 encoded on your desktop. To copy myfile:

```sh
base64 myfile > myfile.b64
```
1. Open the screen session ```screen /dev/ttyUSB0 115200```
2. Type *Ctrl-A* then ```:readreg p /path/to/myfile.b64``` 
3. Then, still in screen, on the target RPi:
```sh
base64 -d > myfile
```
4. Type *Ctrl-D*
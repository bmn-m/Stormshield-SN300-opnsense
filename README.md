# Stormshield-SN300
Stormshield SN300 OPNSense 

![IMG_3546](https://user-images.githubusercontent.com/18091782/201496340-d1e2a905-c35f-41c7-b21e-8972f7859640.JPG)

A nice colleague of mine gave me a Stormshield SN300 a few days ago. 
Since I am a big OPNSense fan, I wanted to install the same on the Stormshield. 
Here and in some YouTube videos, I summarize briefly what was necessary and whether it was worth it. 
It should also be mentioned that Stormshield is a Subsidiary of Airbus Defense & Space Cyber Programmes.

## Departure
Without further ado, I broke the warranty seal to take a look at the device from the inside. This is what i saw:

![IMG_3543](https://user-images.githubusercontent.com/18091782/201493442-4b952d6a-2f44-49d7-8fc4-1d4736ba7d8b.JPG)

Hidden behind the silver cooling fins on the left, the VIA NANO U3500 @ 1000MHz (1 core, 1 thread) CPU. Behind the black cooling fins in the center of the picture, probably the VIA VX900. Behind the black fins on the right, the switch controller and just above it the gigabit ethernet controller that connects the board to the switch. Above, in beautiful green, the RAM with 2GB and below left in blue: The gigantic 2GB flash memory. 
2GB are unfortunately too small for the smallest OPNSense image. It needs at least 3GB.
No less important and probably therefore with two green ok stickers, the AMI BIOS from 2014 in version 02.69. Nice. 

## 1st-Boot
The first thing I did when booting was to press delete to get into the BIOS. Quiet and quick boot switched off, date and time set, saved and rebooted. Short POST image, then nothing. Should not bother us further, we want to "install" OPNSense anyway.  
![POSTImage](https://user-images.githubusercontent.com/18091782/201491974-6cc32e92-4ff2-418d-95e9-075b5f0ee60a.png)

## OPNSense Image
Because the 2GB flash is not big enough, I bought a 128GB 2.5 inch SATA3 SSD for a measly 15€.
As soon as the SSD was delivered, I downloaded the nano image from https://opnsense.org/download/ and unpacked it with 

```bzip2 -dk OPNsense-22.7-OpenSSL-nano-amd64.img.bz2```

Why the nano image? Because we don't have to go through an installation routine and can simply write the image with 

```dd if=OPNsense-22.7-OpenSSL-nano-amd64.img of=/dev/sda```

to the new SSD.
What does nano mean: a preinstalled serial image for USB sticks, SD or CF cards as MBR boot. These images are 3GB in size and automatically adapt to the installed media size after first boot.
Because the SATA port on the board is rather inconveniently located and there is no space on either side, you need a SATA power and data combo cable to connect an SSD.
Connecting... We will find the place in the case for the SSD later on.
Because it was still lying around with me: I upgraded the RAM to 4GB.

![IMG_3557](https://user-images.githubusercontent.com/18091782/201497103-8c0ce131-43e9-4d4c-99a5-883a783cfc7e.JPG)

## 2nd-Boot
After the POST messages OPNSense booted like any other hardware. We are welcomed by the FreeBSD and can log in with the known credentials root and opnsense. That was easy.

## Networking
Opnsense does not detect an active network interface after booting, even though I have plugged a network cable into one of the eight switch ports. Unfortunately, there is also no LED that shows a link. Only one interface and no link, for a firewall rather bad. But this should not stop us in our mission. 
As we have seen on the mainboard, there seems to be a switch chipset. You usually connect to a switch with a serial console, at least for the initial configuration.
On https://docs.freebsd.org/en/books/handbook/serialcomms/ you can find out: Call-out ports are named /dev/cuauN on FreeBSD. it also says that here are at least two utilities in the base-system of FreeBSD that can be used to work through a serial connection: cu and tip. We will use cu.

```cu -l /dev/cuau1 -s 19200```

connects us with the switch.
What the switch can do, i have put here for you
https://github.com/bmn-m/Stormshield-SN300/blob/main/SwitchMenu
We skip all the configuration options and display the switchports.

```port state```
shows 
```
Port  State     
----  --------  
1     Disabled   
2     Disabled  
3     Disabled 
4     Disabled  
5     Disabled   
6     Disabled   
7     Disabled   
8     Disabled  
9     Disabled
```
As you probably already guessed correctly, the switchports should be enabled.

```port state 1-9 enable```
does the trick.

With 

```port mode```
we can see
```
Port  Mode         Link  
----  -----------  ----  
1     Auto         1Gfdx
2     Auto         Down
3     Auto         Down
4     Auto         Down
5     Auto         Down
6     Auto         Down
7     Auto         Down
8     Auto         Down
9     1Gfdx        1Gfdx
```
that probably port one is plugged into the switch and port nine is connected to our internal ethernet controller


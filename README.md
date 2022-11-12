# Stormshield-SN300
Stormshield SN300 OPNSense 

A nice colleague of mine gave me a Stormshield SN300 a few days ago. 
Since I am a big OPNSense fan, I wanted to install the same on the Stormshield. 
Here and in some YouTube videos, I summarize briefly what was necessary and whether it was worth it. 

![grafik](https://user-images.githubusercontent.com/18091782/201491974-6cc32e92-4ff2-418d-95e9-075b5f0ee60a.png)

It should also be mentioned that Stormshield is a Subsidiary of Airbus Defense & Space Cyber Programmes.

# Departure
Without further ado, I broke the warranty seal to take a look at the device from the inside. This is what i saw:

![IMG_3543](https://user-images.githubusercontent.com/18091782/201493442-4b952d6a-2f44-49d7-8fc4-1d4736ba7d8b.JPG)

Hidden behind the silver cooling fins on the left, the VIA NANO U3500 @ 1000MHz (1 core, 1 thread) CPU. Behind the black cooling fins in the center of the picture, probably the GPU. Behind the black fins on the right, the switch controller and just above it the gigabit ethernet controller that connects the board to the switch. Above, in beautiful green, the RAM with 2GB and below left in blue: The gigantic 2GB flash memory. 
2GB are unfortunately too small for the smallest OPNSense image. It needs at least 3GB.
No less important and probably therefore with two green ok stickers, the AMI BIOS from 2010 in version 02.69. Nice. 

# 1st-Boot
The first thing I did when booting was to press delete to get into the BIOS. Quiet and quick boot switched off, date and time set, saved and rebooted. Short POST image, then nothing. Should not bother us further, we want to "install" OPNSense anyway.  


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

Hidden behind the silver cooling fins on the left, the VIA NANO U3500 @ 1000MHz (1 core, 1 thread) CPU. Behind the black cooling fins in the center of the picture, probably the VIA VX900. Behind the black fins on the right, the switch controller and just above it the Intel(R) Gigabit CT 82574L controller, that connects the board to the switch. Above, in beautiful green, the RAM with 2GB and below left in blue: The gigantic 2GB flash memory. 
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

## Switching
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
that probably port one is plugged into the switch and port nine is connected to our internal ethernet controller.
Since we want to use all ports with opnsense we have to resort to VLANs. 
With 

```vlan conf```
we can see
```
VLAN Configuration:
===================


Port  PVID  Frame Type  Ingress Filter  Tx Tag      Port Type      
----  ----  ----------  --------------  ----------  -------------  
1     1     All         Disabled        Untag PVID  Unaware        
2     2     All         Disabled        Untag PVID  Unaware        
3     3     All         Disabled        Untag PVID  Unaware        
4     4     All         Disabled        Untag PVID  Unaware        
5     5     All         Disabled        Untag PVID  Unaware        
6     6     All         Disabled        Untag PVID  Unaware        
7     7     All         Disabled        Untag PVID  Unaware        
8     8     All         Disabled        Untag PVID  Unaware        
9     None  Tagged      Disabled        Untag PVID  C-Port         

VID   VLAN Name                         Ports
----  --------------------------------  -----
1     default                           1,9
2                                       2,9
3                                       3,9
4                                       4,9
5                                       5,9
6                                       6,9
7                                       7,9
8                                       8,9

VID   VLAN Name                         Ports
----  --------------------------------  -----
VLAN forbidden table is empty
```
that each port is already assigned an untagged VLAN except for port nine. Our internal port is a VLAN trunk. Perfect, on the switch we are done for now.  
To exit the switch console you have to enter the following key combination.

```~.```

## Networking
In the next step we have to link the switchports to the OPNSense via the VLANS. This logical link is only an example, but it gives us the greatest possible flexibility.

```
----------------------------------------------
|      Hello, this is OPNsense 22.7          |         @@@@@@@@@@@@@@@
|                                            |        @@@@         @@@@
| Website:	https://opnsense.org/        |         @@@\\\   ///@@@
| Handbook:	https://docs.opnsense.org/   |       ))))))))   ((((((((
| Forums:	https://forum.opnsense.org/  |         @@@///   \\\@@@
| Code:		https://github.com/opnsense  |        @@@@         @@@@
| Twitter:	https://twitter.com/opnsense |         @@@@@@@@@@@@@@@
----------------------------------------------


*** OPNsense.localdomain: OPNsense 22.7.7_1 (amd64/OpenSSL) ***

 LAN (em0) -> v4: 192.168.1.1/24

 HTTPS: SHA256 XX XX XX
 SSH:   SHA256 xxxxxxxx (ECDSA)
 SSH:   SHA256 xxxxxxxx (ED25519)
 SSH:   SHA256 xxxxxxxx (RSA)

  0) Logout                              7) Ping host
  1) Assign interfaces                   8) Shell
  2) Set interface IP address            9) pfTop
  3) Reset the root password            10) Firewall log
  4) Reset to factory defaults          11) Reload all services
  5) Power off system                   12) Update from console
  6) Reboot system                      13) Restore a backup

Enter an option: 1

Do you want to configure LAGGs now? [y/N]: n
Do you want to configure VLANs now? [y/N]: y

WARNING: all existing VLANs will be cleared if you proceed!

Do you want to proceed? [y/N]: y

Valid interfaces are:

em0              00:0d:b4:aa:aa:aa Intel(R) Gigabit CT 82574L

```

Menu-driven, we now assign eight VLAN interfaces to interface em0, the parent interface. The result should look like this: 

```
em0              00:0d:b4:aa:aa:aa Intel(R) Gigabit CT 82574L
em0_vlan1        00:00:00:00:00:00 VLAN tag 1, parent interface em0
em0_vlan2        00:00:00:00:00:00 VLAN tag 2, parent interface em0
em0_vlan3        00:00:00:00:00:00 VLAN tag 3, parent interface em0
em0_vlan4        00:00:00:00:00:00 VLAN tag 4, parent interface em0
em0_vlan5        00:00:00:00:00:00 VLAN tag 5, parent interface em0
em0_vlan6        00:00:00:00:00:00 VLAN tag 6, parent interface em0
em0_vlan7        00:00:00:00:00:00 VLAN tag 7, parent interface em0
em0_vlan8        00:00:00:00:00:00 VLAN tag 8, parent interface em0
```
Now we assign the LAN and the WAN interface, as well as an IP address, so that we can log on to the web interface. For login via web interface a single assignment would be sufficient, but while we are at it...
After the login on the web interface we find the following picture in the menu under Interfaces -> Other Types -> VLAN:

![webgui](https://user-images.githubusercontent.com/18091782/201519306-0bc24536-3d8f-4dcb-b85d-7450a62493a2.png)

## Wireguard
I would like to determine two performance values later. One is the classic routing and packet filter (firewalling) performance and the other is the data throughput when the data is encrypted and sent through a wireguard tunnel. The wireguard management package can be installed via System -> Firmware -> Plugins -> os-wireguard. Since 29.10.2022, the wireguard module is officially available via FreeBSD ports and thus again official part of the FreeBSD kernel, which was not the case for over a year now. https://reviews.freebsd.org/rG4c6c8f51fdb7e2b3870ec5a6fa5dce51ad3b25a5

The VPN configuration will be a classic site to roadwarrior architecture.   
After installing the wireguard package, we can activate wireguard in the menu under VPN -> Wireguard and add a local instance and an endpoint.
For the endpoint we need to create a private public key pair. For the local instance this happens automatically.
Assuming you have already installed the wireguard package on a client with
```
sudo apt install wireguard
```
you can generate the key pair with 

```
wg genkey | tee privatekey | wg pubkey > publickey
```

# ipfire_on_pcengines_apu2c4-
A HowTo install IPFire on a pc engines apu2c4 (most probably apu2e4 and apu2e2 will be similar or the same)

This Guide is under GPL3 license, see [LICENSE](/LICENSE)

## Introduction

In this tutorial i will describe all steps I took to get IPFire running on my PCEngins apu2c4  
It will include:
- SSH with Public Key Authentication
- dyndns (zoneedit)
- IPsec and/or OpenVPN
- SMB (Samba/CIFS/Windows share)
- Transmission (a BitTorrent Client running 24/7 which helps to shar Linux ISO Images etc.)
- External Harddisk (ext4) for SMB/Transmission
- Automatic Backup on a second harddisk (NTFS) as soon as it gets connected (low tech version)
- MC (midnight commander)

Yes, I know it's kinda stupid running SMB on your router. Shoot me. I just use it for storing things that need zero security.  

## Abreviations
APU = the APU.2C4 system board


# The hardware
Ordered from <pcengines.ch>

```
1       apu2c4          APU.2C4 system board 4GB
1       case1d2bluu     Enclosure 3 LAN, blue, USB
1       ac12veur2       AC adapter 12V 2A euro for IT equipment
1       msata16d        SSD M-Sata 16GB MLC Phison
1       wle200nx        Compex WLE200NX miniPCI express card
2       pigsma          Cable I-PEX -> reverse SMA
2       antsmadb        Antenna reverse SMA dual band
1       spi1a           Flash recovery adapter apu2
```

wle200nx, pigsma and antsmadb are optional if you want WLAN (used by me as WLAN HotSpot)
spi1a probably not needed, just for aditional saftey if you intend to brick the APU ;-)

Furthermore you need 
- a Computer (I used a Win10 Laptop - there I said it, the W Word)
- a RS232 serial null modem cable  (db9cab1 if you want to order it also from pcengines)
- most probably a Serial to USB Converter (unless you have a machine with RS232 who has that these days)

# Assembly
not documentd, sorry. Maybe later.

# Cabeling
If you look at the APU from behind, from left to right I have
1. RS232
2. GREEN (LAN)
3. RED (WAN)
4. ORANGE (DMZ)
5. USB Top: external disc (ext4) for SMB and as a storage for Transmission (running 24/7)
   USB Bottom: external disc (NTFS) for backing up SMB
6. Power

# Installation
According to https://pcengines.ch/howto.htm#OS_installation you'll have to:

boot via iPXE and get the installation image directly from IPfire via HTTP without the need of preparing a bootable storage device.
These are the steps:
- enable network boot in the apu BIOS setup
- boot the apu and press CTRL-B to get into the iPXE console
- receive an IP address from your local DHCP Server by typing: ifconf
- type: chain http://boot.ipfire.org/releases/ipfire-boot/latest/ipxe.kpxe
- install IPfire with the serial console option

So let's try this.
- Conect rs232 Cable to your PC/USB Converter and figure out which com port it is running (devicemanager is your friend)
- fire up putty, Serial, COMx / 115200

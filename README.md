# ipfire_on_pcengines_apu2c4-
A HowTo install IPFire on a pc engines apu2c4 (most probably apu2e4 and apu2e2 will be similar or the same)

This Guide is under GPL3 license, see [LICENSE](/LICENSE)

## Introduction

In this tutorial i will describe all steps I took to get IPFire running on my PCEngins apu2c4  
It will include:
- SSH with Public Key Authentication
- SMB (Samba/CIFS/Windows share)
- Transmission (a BitTorrent Client running 24/7 which helps to shar Linux ISO Images etc.)
- MC (midnight commander)
- External Harddisk (ext4) for SMB/Transmission
- Automatic Backup on a second harddisk (NTFS) as soon as it gets connected (low tech version)
- dyndns (zoneedit)
- IPsec and/or OpenVPN

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
- PuTTY
- A USB Stick (for updating the BIOS)
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

# coreboot / BIOS update
Windows: download TinyCore USB installer v1.8 from https://pcengines.ch/howto.htm#TinyCoreLinux
- Start Installer and prepare USB Stick **CAUTION all data on stick is LOST**
- Download Images from https://pcengines.github.io/ copy an unzip them to USB Stick
- Connect rs232 Cable to your PC/USB Converter and figure out which com port it is running (devicemanager is your friend)
- on the PC start putty, Serial, COMx / 115200
- Plug in the USB drive and power up the APU
- Press F10 for Boot Menu
- Press 2(?) for setup
- Press a(?) for moving USB to first place
- Press u to enable USB boot
- Press s to save&exit
- Wait and see it booting from USB you shoud see
```
  ____   ____   _____             _
 |  _ \ / ___| | ____|_ __   __ _(_)_ __   ___  ___
 | |_) | |     |  _| | '_ \ / _` | | '_ \ / _ \/ __|
 |  __/| |___  | |___| | | | (_| | | | | |  __/\__ \
 |_|    \____| |_____|_| |_|\__, |_|_| |_|\___||___/
                            |___/

TinyCore 6.4 www.tinycorelinux.com

login[649]: root login on 'ttyS0'
waiting for USB being mounted .sd 4:0:0:0: [sdb] No Caching mode page found
..

Welcome to TinyCore running on PC Engines apu boards
To update the BIOS type "flashrom -w apu_xyz.rom -p internal"

[+52.4 C][root@box:/media/TINYCORE]$
```
- type `ls -la` to find your rom
- type `"flashrom -w /path/to/image.rom -p internal` to flash your rom
If you get an error like this:
```
flashrom v0.9.9-r1954-beead91-17 on Linux 4.2.9-tinycore (i686)
flashrom is free software, get the source code at https://flashrom.org

coreboot table found at 0xcffd6000.
Found chipset "AMD FCH".
Enabling flash write... OK.
Identifying board "PC Engines apu2"... OK.
Found Winbond flash chip "W25Q64.V" (8192 kB, SPI) mapped at physical address 0xff800000.
This coreboot image (PC Engines:apu2) does not appear to
be correct for the detected mainboard (PC Engines:PC Engines apu2).
Aborting. You can override this with -p internal:boardmismatch=force.
```
you may have to use force ;-)

- this may take some time you should see something along:

```
flashrom v0.9.9-r1954-beead91-17 on Linux 4.2.9-tinycore (i686)
flashrom is free software, get the source code at https://flashrom.org

coreboot table found at 0xcffd6000.
Found chipset "AMD FCH".
Enabling flash write... OK.
Identifying board "PC Engines apu2"... OK.
Found Winbond flash chip "W25Q64.V" (8192 kB, SPI) mapped at physical address 0xff800000.
This coreboot image (PC Engines:apu2) does not appear to
be correct for the detected mainboard (PC Engines:PC Engines apu2).
Proceeding anyway because user forced us to.
Reading old flash chip contents... done.
Erasing and writing flash chip... Erase/write done.
Verifying flash... VERIFIED.
```
- type `reboot` to reboot
- do a full cycle until you are on the TinyCore prompt again
- type `reboot` to reboot but this time press F10
- Press 2/3(?) for setup
- Press c(?) for moving mSata to first place
- Press u to disable USB boot
- Press s to save&exit


# Installation PXE
**CAUTION PXE currently does not seem to work jump to Installation USB**

According to https://pcengines.ch/howto.htm#OS_installation you'll have to:

boot via iPXE and get the installation image directly from IPfire via HTTP without the need of preparing a bootable storage device.
These are the steps:
- enable network boot in the apu BIOS setup
- boot the apu and press CTRL-B to get into the iPXE console
- receive an IP address from your local DHCP Server by typing: ifconf
- type: chain http://boot.ipfire.org/releases/ipfire-boot/latest/ipxe.kpxe
- install IPfire with the serial console option

So let's try this:
- Connect the APUs RED Interface to the internet (middle ethernet port)
- Connect rs232 Cable to your PC/USB Converter and figure out which com port it is running (devicemanager is your friend)
- fire up putty, Serial, COMx / 115200
- Press F10 for Boot Menu
- Press 2(?) for setup
- Press f(?) for moving iPXE to first place
- Press n to enable PXE boot
- Press s to save&exit
- Wait and see it booting press N for network boot
- type cursor up and return for ipxe shell
- type `ifconf` to configure interfaces
- type `dhcp` to get an IP from DHCP
- type: `chain http://boot.ipfire.org/releases/ipfire-boot/latest/ipxe.kpxe` (sorry, copy paste does not seem to work here)
- wait....

=> it does not work

# Installation from USB
So let's try the USB way:
- Install Rufus from https://rufus.ie/
- download the ISO Image from https://www.ipfire.org/download/ipfire-2.25-core153
- Plugin USB and open Open Rufus
- Choose correct USB Stick **CAUTION all data on stick is LOST**
- Choose the iso image, dont change any of the other settings and click Start (agree with everythign)
- Connect the APUs RED Interface to the internet (middle ethernet port)
- Plug in the USB drive into the APU and power it up
- Press F10 for Boot Menu
- Press 3(?) for setup
- Press a(?) for moving USB to first place
- Press u to enable USB boot
- make the putty window 80x24
- Press s to save&exit
- Wait and see it booting, choose serial console installation / unatended 
 (maybe tricky as putty does not seem to work properly, maybe switch of xon/xoff in putty? if you do so, remember to enable xon/xoff afterwards)
 (maybe adding `console=ttyS0,115200n8` in grub console would help)
- Choose english
- Choose Start Instalation
- Choose Download Installation Image (why on earth....)
- Wait, wait, wait
- Tab and Space to accept license, tab and return to continue
- Choose the SATA_SSD disk (probably sda)
- Choose **Delete all data**
- Choose ext4
- Wait for installation to finish
- Choose reboot
- Press F10 for Boot Menu
- Press 3(?) for setup
- Press c(?) for moving mSATA to first place
- Press u to **disable USB boot**
- Possibly press n to **disable PXE boot**
- Press s to save&exit
- Now it should boot into ipfire
- Choose your keyboard (`de_CH-latin1` for switzerland)
- Choose Timezone 
- Choose Name (ipfire)
- choose domain (raudi)
- choose password  for root (console / serial)
- choose password for admin (webinterface) 
- choose "Drivers and card assignments" "Network configuration type" and "GREEN + RED + ORANGE + BLUE"
- choose "Assigned Cards", asign GREEN RED ORANGE to topmost interface, assign blue to Wireless
- choose " Address settings"
  - assign GREEN  to 192.168.xxx.1 (with xxx any number 1 to 128) Networkmask 255.255.255.0
  - assign BLUE   to 192.168.xxx.1 (with xxx any different number 1 to 128) Networkmask 255.255.255.0
  - assign ORANGE to 192.168.xxx.1 (with xxx any different number 1 to 128) Networkmask 255.255.255.0
  - set RED to DHCP, `ipfire`
- continue with DONE (youmust not set "Gateway settings")
- enable DHCP 
```
           │  Start address:                  192.168.37.100____   │
           │  End address:                    192.168.37.255____   │
           │  Primary DNS:                    192.168.37.1______   │
           │  Secondary DNS:                  __________________   │
           │  Default lease (mins):           60________________   │
           │  Max lease (mins):               1440______________   │
           │  Domain name suffix:             ipfire.raudi______   │
```
- OK and OK to complete
- let it start for the first time (this may take some time)
- login with root/password
- Well done!
- now do a `shutdown now`
- Connect your Computer via LAN to GREEN (probably with a switch)
- Power on the APU
- check that it is starting with putty
- once it is up check if your PC get's an IP within the correct range
- browse to https://ipfire:444 (Yes, for now you have to accept the security risk)
- Make sure RED is connected (System/Home)
- Yeeeehaw it's working

# SSH with Public Key Authentication
# SMB (Samba/CIFS/Windows share)
# Transmission (a BitTorrent Client running 24/7 which helps to shar Linux ISO Images etc.)
# MC (midnight commander)
# External Harddisk (ext4) for SMB/Transmission
# Automatic Backup on a second harddisk (NTFS) as soon as it gets connected (low tech version)
# dyndns (zoneedit)
# IPsec and/or OpenVPN



# other notes







# other notes
installed packages
```
- avahi
- cups-filters
- cups
- dbus
- ghostscript
- hostapd
- krb5
- libdaemon
- libtiff
- libtirpc
- mc
- nano
- perl-Parse-Yapp
- rsync
- samba
- transmission
```

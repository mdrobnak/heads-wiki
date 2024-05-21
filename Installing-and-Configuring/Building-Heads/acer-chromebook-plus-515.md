---
layout: default
title: Acer Chromebook Plus 515 (CP515-2H) (Google Omnigul)
permalink: /acer-chromebook-plus-515/
nav_order: 1
parent: Step 1 - Building Heads
grand_parent: Installing and configuring
---

Acer Chromebook Plus 515 (CP515-2H) (Google Omnigul)
====

This is a reasonably powerful low-cost Chromebook Plus. It has the following specifications:
* 8 GB RAM
* 12th Gen Intel Core i3-1215U
* UHD Graphics
* Intel AX211 Wi-Fi 6E
* 32MB Winbond chip - W25Q256JV\_M
* 128 GB UFS Storage [00:12.7 in lspci output]

It comes from Google with a Coreboot BIOS, making it a suitable target for Heads. The payload is depthcharge, which is used to load ChromeOS. It has a reasonable amount of security features out the box, which requires us to remove them before proceeding.

At this time, the Management Engine is in a soft-disabled state. The ME on this Chromebook uses the "Lite" SKU, and the HAP (High Assurance Platform) bit for it has not been found. In addition, it uses a dual-partition (Read/Write and Read Only) setup, and there is currently no way to open up the Read Only partition at this time, so even if you flash the firmware diescriptor as described below, there will still be a region of Flash that is inaccessible.

Prerequisites
-----
Follow the [Chultrabook Docs](https://docs.chrultrabook.com/docs/firmware/flashing-firmware.html) which will guide you through the initial setup. The Write Protect removal method to use here is to disconnect the battery. See the Lifecycle Extension Guide below for disassembly instructions.
Flash the UEFI firmware.
Unlike on other platforms, there are more than the Firwmare Descriptor, BIOS, and ME in the Flash layout. There is VPD, or Vital Product Data which contains things such as your serial number and WiFi MAC address. This is important to not break.


Building Heads
-----
Follow the instructions on the main page to set up the Docker container, and use `make BOARD=omnigul` to build the ROM image.


Initial Flash of Heads
-----
In theory you can use a [SuzyQ cable to flash](https://wiki.mrchromebox.tech/Unbricking#Unbricking.2FFlashing_with_a_Suzy-Q_cable) the entire chip but the author was unable to get this to work.

Instead, follow the [Lifecycle Extension Guide](https://global-download.acer.com/GDFiles/Document/Lifecycle%20Ext.%20Guide/Lifecycle%20Ext.%20Guide_Acer_1.0_A_A.pdf?acerid=638320940119709511&Step1=&Step2=&Step3=CB515-2H&OS=ALL&LC=en&BC=ACER&SC=PA_6) to disassemble the main parts of the laptop. Remove the battery, the WiFi antenna cables, the Display cable, tradckpad cable and other ribbon cables connected to the mainboard. Remove the screws on the fan, and on the latch which is over the mainboard. Remove the screw from the mainboard near the edge. After that, the mainbaord will be freed. Turn it over and you will see the Winbond flash chip.

<Insert Images>

Flash the Flash Descriptor by holding the probe in place, and running flashrom.
<Insert Commands>



OS Installation
-----
If installing Qubes 4.2, there is a point in the installation in which the kernel will crash. There's a workaround, thankfully.

Copy the following to a USB stick:

Name this run.sh:
```
#!/bin/sh
until [ -d /mnt/sysimage/etc/dracut.conf.d ]
do
	sleep 1
done
cp -p ufshcd-driver.conf /mnt/sysimage/etc/dracut.conf.d/

until [ -d /mnt/sysimage/usr/lib/dracut/modules.d/90kernel-modules ]
do
	sleep 1
done
cp -p module-setup.sh /mnt/sysimage/usr/lib/dracut/modules.d/90kernel-modules/
```

Name this ufshcd-driver.conf:
```
add_drivers="ufshcd_core ufshcd_pci"
```

For the modules file...this is too large for this wiki.

Appendix
-----
Coreboot Version: MrChromeBox's fork, currently at version 4.22.4.
Linux Version: 6.6.30, 6.1.90 also tested.
Base Coreboot config: Config from above. Modifications: Linux payload, HECI (Management Engine) disable.
Base Linux config: linux-nitropad-x.config
Base Board config: nitropad-nv41.conf

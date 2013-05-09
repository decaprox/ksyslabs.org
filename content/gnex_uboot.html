---
title: Samsung Galaxy Nexus U-Boot
---

Introduction
====================

This page describes the port of the u-boot bootloader to the Galaxy Nexus phone

Rationale
====================

There were a couple reasons to port u-boot to Galaxy Nexus

* Security: we cannot trust the proprietary samsung bootloader
* Implementing dual-boot for original and custom firmware
* Booting Genode operating system

Demo
====================

Compilation from source
---------------------

Source code is in [https://github.com/Ksys-labs/uboot-tuna](https://github.com/Ksys-labs/uboot-tuna)

There exist two branches of interest

* master - contains the official stable releases. may be force-pushed and rebased, beware
* tuna-fosdem-hacks contains the u-boot that was used for FOSDEM 2013 to demo booting Genode

To compile, you need to have the ARM cross-compiler. I recommend codesourcery 2010q1-188 because that' s what I 'm using and some users reported that newer compilers produce broken binaries.

There are two ways to use the u-boot. One is flashing it instead of the Samsung SBL bootloader. The other one is chainloading it from the SBL.

Flashing instead of SBL has the following advantages

* Faster boot time than chainloading
* Ability to use the standard partitioning layout

There is a number of issues and therefore we do not recommend flashing it instead of SBL

* No Fastboot support (preliminary USB RNDIS and DHCP BOOTP support is available), you' ll have to use OMAPFlash to restore the device if you flash a non - working kernel
* No display initialization.You 'll have to disable the “Check for Bootloader initialization” option in kernel config

By default, the chainloaded version is compiled. It is loaded (by the SBL) to the address **0x81808000**.

If you want to build the SBL replacement version, edit the **include/configs/omap4_tuna.h** file and uncomment the **#define TUNA_SPL_BUILD line**. X-loader loads the bootloader to the address **0xa0208000**.

> export PATH=/home/alexander/handhelds/armv6/codesourcery/bin:$PATH
> export ARCH=arm
> export CROSS_COMPILE=arm-none-eabi-

> U_BOARD=omap4_tuna
> make clean
> make distclean

> make ${U_BOARD}_config
> make -j8 ${U_BOARD}
> mkbootimg --kernel u-boot.bin --ramdisk /dev/null -o u-boot.aimg


Installation
====================

Chainloaded Mode

You will need the root access to your device.
You can take the prebuilt u-boot here. [gnex-uboot-chainloaded.img]

The u - boot has the support for android boot images.When flashed instead of the SBL, it boots the kernel off the "Boot" partition. When chainloaded, it looks for the kernel in **/system/boot/vmlinux.uimg**.
Additionally, it first looks for the **/system/boot/boot.scr.uimg** so you can put custom commands there and override the kernel image. 

It also supports booting custom images from **/sdcard/boot/vmlinux.uimg** and **/sdcard/boot/boot.scr.uimg**. 

If you need larger images, I suggest that you use the **tuna-fosdem-hacks** branch, format the cache partition to ext2 and put the files to **/cache/media/boot/**. 

push the files to your device via adb 

> adb push gnex-uboot-chainloaded.img /sdcard/ 
> adb hell 

now, in the device shell, do the following:

> su
> cat /dev/block/platform /omap/omap_hsmmc.0/by-name/boot > /sdcard /vmlinux.uimg 
> mount - o remount, rw /system mkdir/system/boot 
> cp /sdcard/vmlinux.uimg /system/boot/ 
> cat /sdcard/gnex-uboot-chainloaded.img > /dev/block/platform /omap /omap_hsmmc.0/by-name/boot 
> sync
> reboot 

you can also use fastboot to flash u-boot.img instead of dd'ing it

> fastboot flash:raw boot u-boot.img"

Replacing samsung bootloader
====================

OMAP4 devices cannot be bricked completely because the CPU has a firmware loader in the OTP (one-time programmable) memory. When the device is powered, it tries booting from USB.

Make sure to have an old version of x-loader (PRIMEKK14) because newer ones have the security hole which allowed booting unsigned bootloaders fixed.

Take the x-loader here [gnex-xloader-working.img]

> adb push gnex-xloader-working.img /sdcard/

The installation procedure is roughly the same, but use sbl partition. Also, install xloader by

> cat /sdcard/gnex-xloader-working.img > /dev/block/platform/omap/omap_hsmmc.0/by-name/xloader

There exists a Samsung recovery tool which can unbrick the devices with corrupted xloader/SBL. You will need a computer running Windows XP.

Search the internet for the archive named “OMAPFlash_tuna.zip” which has md5 “ddbf07a1d36b044c40af5788a83b5395”. We cannot upload it here because of the unclear license status.

Making images
====================

You can either use Android' s mkbootimg to produce ANDROID !type images (not recommended) or u-boot's mkimage (in the u-boot tools directory) to make boot images. Using ANDROID! format is discouraged because the loader code in the u-boot is buggy and may fail in some corner cases such as large images.

making a custom boot image
---------------------

> mkimage -A arm -O linux -T kernel -C none -a 0x80008000 -e 0x80008000 -n linux -d zImage vmlinux.uimg

> \#alternatively, just do that when compiling linux
> 
> \#do not forget to add mkimage to your PATH variable
> make uImage
> making a custom boot script

> mkimage -A arm -O linux -T script -C none -a 0x84000000 -e 0x84000000 -n android -d boot.scr boot.scr.uimg

Booting Modes
====================

The bootloader supports several boot modes. Each boot mode is indicated by the color of the LED and activated by a combination of hardware buttons. It also supports the Android “reboot to recovery” and “reboot to bootloader” features

* Normal Boot -> no keys are pressed, cyan LED
* Recovery Boot -> Volume Up key pressed, green LED
* Custom Boot -> Volume Down key pressed, blue LED
* USB RNDIS mode -> both Volume keys pressed, purple LED

Pitfalls
====================

* No Fastboot or DFU (RNDIS BOOTP is untested) -> not a big deal if you' re chainloading, right ?
* Serial number is always 0123456789 abcdef or sth like that.Anyone to fix that ?
* UART support is quirky.The device will likely hang if booted with the UART cable.Workaround : boot without the UART cable and plug right after the purple LED flashes.

A sample boot script for android
====================

Make a boot.scr.uimg from it and push it to the correct location.

> setenv bootargs "mem=1G vmalloc=768M omap_wdt.timer_margin=30 mms_ts.panel_id=18 no_console_suspend console=ttyFIQ0";
> 
> setenv loaddaddr 0x82000000;
> 
> setenv devtype mmc;
> 
> setenv devnum 0;
> 
> setenv kernel_part 0xc;
> 
> setenv kernel_name /media/boot/vmlinux.uimg;
> 
> echo Load Address: ${loaddaddr};
> 
> echo cmdline:${bootargs};
> 
> if ext4load ${devtype} ${devnum}:${kernel_part} ${loaddaddr} ${kernel_name}; then
> 
> 	bootm ${loaddaddr};
> 
> 	exit 0\;
> 
> elif ext2load ${devtype} ${devnum}:${kernel_part} ${loaddaddr} ${kernel_name}; then
> 
> 	bootm ${loaddaddr};
> 
> 	exit 0;
> 
> else

> 	echo failed to boot custom image;

> fi

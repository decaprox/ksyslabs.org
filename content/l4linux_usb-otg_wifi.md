---
title: OMAP3 L4Linux usb-otg with wifi 
---

Overview
====================

L4Reap is a modified version of L4Linux created to run on TI OMAP3 (Gumstix Overo, IGEP). The goal of this project is to make a certain number of hardware resources accessible and usable from whithin a paravirtualized environment.

List of Supported Hardware
---------------------

* Libertas WiFi module
* USB OTG Ethernet Gadget

Build Instructions
====================

Cross Toolchain
---------------------

L4Reap is known to work with CodeSourcery 2011.03 toolchain for ARM:

> arm-2011.03-41-arm-none-linux-gnueabi-i686-pc-linux-gnu.tar.bz2

We assume that toolchain is set up and available as arm-linux-gcc, arm-linux-ld, etc.

Setup
---------------------

> mkdir build
> export L4DIR=`pwd`
> export L4BUI=$L4DIR/build 
> svn co http://svn.tudos.org/repos/oc/tudos/trunk/@40 src
> git clone git://github.com/Ksys-labs/L4Reap.git

Building
---------------------

Building Fiasco.OC Kernel

> make BUILDDIR=$L4BUI/kernel -C $L4DIR/src/kernel/fiasco
> make -C $L4BUI/kernel config

Choose following options from menu:

> Target configuration  ---> 
>      Architecture (ARM processor family)  --->
>      Platform (TI OMAP)  -->
>      OMAP Platform (Beagle Board)  --->
> make O=$L4BUI/kernel -C $L4DIR/src/kernel/fiasco

Build L4Re

> mkdir $L4BUI/l4
> make O=$L4BUI/l4 -C $L4DIR/src/l4 config

Choose following options from menu:

>  Target Architecture (ARM architecture)  --->                            
>     CPU variant (ARMv7A type CPU)  --->
>     Platform Selection (Beagleboard)  --->   
>  make O=$L4BUI/l4 -C $L4DIR/src/l4

Build L4Linux

> make -C L4Reap/l4linux O=$L4BUI/l4reap L4ARCH=arm CROSS_COMPILE=arm-linux- l4reap_defconfig
> echo CONFIG_L4_OBJ_TREE=\"$L4BUI/l4\" >> $L4BUI/l4reap/.config
> make -C L4Reap/l4linux O=$L4BUI/l4reap L4ARCH=arm CROSS_COMPILE=arm-linux-

Creating a Bootable Image
---------------------

>  make O=$L4BUI/l4 -C $L4DIR/src/l4 \
>      MODULES_LIST=$L4DIR/L4Reap/files/modules.list \
>      MODULE_SEARCH_PATH=$L4DIR/L4Reap/files:$L4BUI/kernel:$L4BUI/l4reap \
>      E=l4reap uimage

Running
====================

### Booting from MMC:

> mmc rescan 0
> fatload mmc 0 80000000 l4img
> bootm 80000000

Bugs and Limitations
====================

The system does not survive:

* OTG unplugging
* ifconfig usb0 down
* Hi, I'm a new item!

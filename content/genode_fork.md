---
title: Genode fork
---

We have the following things in our [repo](https://github.com/Ksys-labs/genode) which aren't moved to Genode upstream yet:

* ### Gumstix Overo platform, OMAP3 framebuffer driver, ADS7846 touchscreen driver 
> We used to use Gumstix board for the first prototype our device. This board is OMAP3 based and it allowed us to use Fiasco.OC and Genode for Gumstix. We have implemented support of framebuffer and touchscreen. Then the platform was changed to OMAP4 and we cancelled a support this code. This code requires a little improvement.
* ### Ts_input adapter 
> This service implement translation raw data from resistive touchscreen to screen coordinates for Nitpicker. Also Ts_input can be used for touchscreen calibration. Ts_input configuration is compatible with tslib.
* ### OMAP4 McSPI driver and SPI session interface 
> This is simple SPI driver for OMAP4. We have implemented only part of McSPI functions and this driver have used only for the TSC2046 touchscreen driver.
* ### TSC2046 touchscreen controller driver 
> Touchscreen driver for the Chipsee expansion board for the Pandaboard.
* ### Qt ssl fix 
> We have done modification QtNetwork for work over SSL connections in qt-based browsers. Also, fixed loading of openssl.conf for using OpenSSL engines in Qt applications.
* ### QtSql and sqlite3 
> Added QtSql library with sqlite3 support.
* ### [Qupzilla](http://www.qupzilla.com/) browser port
* ### Smartcard session interface and driver for TDA8029 smartcard reader 
> This interface and driver implement work with smartcards. TDA8029 reader works over UART.
* ### Virtual network between l4linux's 
> L4linux driver which is implemented the NIC service and it can be used for connect two l4linux instances using the NIC interface. More information is available at Gateway page.
* ### Terminalmux 
> Terminal multiplexer. More information available at [README](https://github.com/Ksys-labs/genode/blob/staging/os/src/server/terminalmux/README)
* ### TWL6030 Voltage Regulator driver 
> This allows to enable various hardware on OMAP4 SoC.
> The code is in **os/include/regulator_session** and **os/src/drivers/regulator**.
* ### OMAP4 I2C driver
* ### I.MX53 I2C driver

We are currently working on the following features:

* ### OMAP4 USB OTG (Client mode for Mentor Graphics MUSB driver).
> We currently have the controller initializing, but the data endpoints are not working.
> The branch is omap4-otg-dirty ; [git](https://github.com/astarasikov/genode.git).
* ### Melfas MMS144 Touchscreen 
> This is the touchscreen used in the Samsung Galaxy Nexus phone.
> A prototype driver and board support code is available in branch tuna-hacks ; Ksys Labs git .
* ### OMAP4 GPIO MUX driver 
> This allows to set up GPIO alternate functions.
> The code is in os/include/platform/omap4/gpiomux_session and os/src/drivers/gpio/omap4_mux.
> The branch is **tuna-hacks**; [Ksys Labs git](https://github.com/Ksys-labs/genode.git).

Below is the list of features that we want to implement but have not yet started. While implementing some hardware drivers (PWM, Clock) it will be required to also design a generic driver session interface for Genode.

* ### EFI GPT partition support 
> This is needed for most new embedded hardware as it is a part of EMMC standard. Also will be needed to support new UEFI laptops
* ### File System DDEKit 
> Currently the choice of filesystems on Genode is limited to VFAT. Providing the support for linux filesystem drivers via DDEKit will allow to always have the latest version of EXT4 driver and access linux partitions without the risk of destroying them.
* ### FS → Block adapter 
> This will allow to present a file in a filesystem as a block device in the same way as loop devices work on *NIX. It is needed to support multiple instances of l4linux without repartitioning the storage device
* ### OMAP4 Clock driver 
> It is required to provide initialization of most peripherals.
* ### OMAP4 DVFS 
> To reduce power consumption
* ### OMAP4 DSS/DSI display 
> Support for the cold initialization of the DSS subsystem and the VC interface to support LCD panels on Samsung Galaxy Nexus and Samsung P5100
* ### Power Management 
> Enhance the sheduler to support idle power saving and (hopefully) suspend2ram.
* ### OMAP4 PWM 
> Mainly for controlling backlight and vibrator motor in the phone.
* ### TWL6040 Sound Codec + Routing 
> Sound support on PandaBoard and Galaxy Nexus

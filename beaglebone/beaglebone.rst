================
BeagleBone Black
================

AM355x Sitara Technical reference:
http://www.ti.com/lit/ug/spruh73l/spruh73l.pdf

System Reference manual:
https://github.com/CircuitCo/BeagleBone-Black/blob/master/BBB_SRM.pdf?raw=true

Schematics (rev C):
https://github.com/CircuitCo/BeagleBone-Black/blob/master/BBB_SCH.pdf?raw=true

U-Boot
======

Source code: git://git.denx.de/u-boot-ti.git

`${bootcount}` works.

Trigger reset from uboot::

  mw.l 0x44E00F00 1 # trigger reset by writing to PRM_RSTCTRL

See *8.1.13.5 PRM_DEVICE Registers* in reference manual

AM355x U-Boot
-------------

U-Boot split in MLO (fist stage uboot) and rest of uboot due to
limited space in internal RAM. Bootstrap loader (IPL - Initial Program
Loader) loads SPL (Secondary Program Loader - MLO in this case). SPL
loads the rest of uboot.

http://processors.wiki.ti.com/index.php/AM335x_U-Boot_User's_Guide

eMMC
----

SPL looked for at offsets `#0`, `#256`, `#512`, `#768`, `#1024` (see
*26.1.7.5.5 MMC/SD Read Sector Procedure in Raw Mode*) U-Boot starts
at `0x400`.

Load MLO and U-Boot from SD card and write to MMC::

  U-Boot # mmc dev 0
  U-Boot # mmc rescan
  U-Boot # mmc dev 1
  U-Boot # fatload mmc 0 ${loadaddr} MLO
  U-Boot # mmc write ${loadaddr} 0x100 0x100
  U-Boot # mmc write ${loadaddr} 0x200 0x100
  U-Boot # fatload mmc 0 ${loadaddr} u-boot.img
  U-Boot # mmc write ${loadaddr} 0x300 0x400

U-Boot environment size (`include/configs/imap3_beagle.h`)::

  /* Always 128 KiB env size */
  #define CONFIG_ENV_SIZE			(128 << 10)

Usable `/etc/fw_env.config`::

  /dev/mmcblk0boot1       0x00000         0x20000
  /dev/mmcblk0boot1       0x20000         0x40000

U-Boot enviroment partitions are read-only by default. To disable RO::

  echo 0 > /sys/block/mmcblk0boot1/force_ro


Linux
=====

Works with kernels:
- mainline
- linux-yocto (3.14.11?)
- linux-ti http://git.ti.com/ti-linux-kernel/ti-linux-kernel

I/O
---

Pinmux offsets in *9.3.1 CONTROL_MODULE Registers* in reference
manual.

Leds
----

See `/sys/devices/leds/`.

Yocto
=====

Machine setting::
   MACHINE = "beaglebone"

Use `meta-ti`. **NOTE**: 3.14 kernel has IPv6 disabled, will trigger
issues with `systemd-networkd`.

As of writing this text the current U-Boot is::

  U-Boot 2014.07 (Jun 01 2015 - 13:54:58)

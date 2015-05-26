================
BeagleBone Black
================

AM355x Sitara Technical reference:
http://www.ti.com/lit/ug/spruh73l/spruh73l.pdf


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

Yocto
=====

::
   MACHINE = "beaglebone"

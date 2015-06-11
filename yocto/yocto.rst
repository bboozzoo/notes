========
Yocto/OE
========

Bitbake reference:
http://www.yoctoproject.org/docs/current/bitbake-user-manual/bitbake-user-manual.html

Developer's manual:
https://www.yoctoproject.org/docs/current/dev-manual/dev-manual.html

Reference manual:
https://www.yoctoproject.org/docs/current/ref-manual/ref-manual.html


site.conf
---------

Override directory locations, ex::

  SCONF_VERSION = "1"
  DL_DIR = "/home/mborzecki/yocto/downloads"
  SSTATE_DIR = "/home/mborzecki/yocto/sstate-cache"
  RM_OLD_IMAGES = "1"


Recipes
-------

*NOTE*: always set `S`!!

Disable certain recipes
+++++++++++++++++++++++

Set `BBMASK` to disable certain recipes. The recipes are not even
parsed. Note, `BBMASK` is a single regexp, must match all masked
recipes.

BBMASK ?= ".*/meta-ti/recipes-misc/(images|payload)/"

wic
---

Creating images::

  wic create sdimage-part -e core-image-minimal -D -o /tmp/wic-image


Build name
----------

To put a build name in `/etc/version` set `BUILDNAME` in on of the
cont files (possibly `local.comf`, or `auto.conf` if triggered from
Jenkins).

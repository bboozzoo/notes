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

g-i-r
+++++

**NOTE**: OE packages have introspection disabled!.

Does not work when cross-compiled. Use meta-gir_.

.. _meta-gir: https://github.com/meta-gir/meta-gir

Vala
++++

Most VAPIs shipped with Vala out of the box. VAPI is generated based
on g-i-r output, not usable when doing cross-compilation. Possible
solution:

- build package on the host
- generate `*.vapi` and `*.deps`
- ship VAPI files with `*.bbappend*`, install files into
  `${D}${datadir}/vala/vapi`


wic
---

Creating images::

  wic create sdimage-part -e core-image-minimal -D -o /tmp/wic-image


Build name
----------

To put a build name in `/etc/version` set `BUILDNAME` in on of the
cont files (possibly `local.comf`, or `auto.conf` if triggered from
Jenkins).

kernel
------

Config fragments
++++++++++++++++

For kernels that do not support config fragments, use *Poor Man's*
config fragment support::

  FILESEXTRAPATHS_prepend := "${THISDIR}/${PN}-${PV}:"
  # reorder *.cfg files as needed
  SRC_URI += "\
        file://usbserial.cfg \
        file://bridge.cfg \
	"

  do_configure_prepend() {

    cfgs="${@ ' '.join([n for n in src_patches(d, True) if n.endswith('.cfg')])}"

    bbnote "configs: ${cfgs}"
    if [ -n "${cfgs}" ]; then
        for cfg in "${cfgs}"; do
            bbnote "Applying config ${cfg}"
            cat ${cfg} >> ${WORKDIR}/defconfig
        done
    fi
  }



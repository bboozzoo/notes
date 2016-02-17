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


Erlang
------

See `meta-erlang`. Recipe::

  inherit erlang
  # if uses rebar
  inherit rebar

Rebar
+++++

Fetch deps before building::

  do_rebar_deps() {
      cd ${S}
      ./rebar get-deps
  }

  do_rebar_deps[deptask] = "do_populate_sysroot"
  addtask rebar_deps after do_unpack before do_patch

Ports
+++++

Ports will require `-lerl_interface -lei`. Broken when running with
`rebar` set ``ERL_EI_LIBDIR` otherwise it will pick a native libdir::

  do_compile() {
      ei_libdir=$(echo ${STAGING_LIBDIR}/erlang/lib/erl_interface-*/lib)
      ERL_EI_LIBDIR=$ei_libdir ./rebar compile
  }

`rebar` is broken wrt. cross compilation, it's not possible to
override `ERL_LDFLAGS`. It gets overwritten with the default
always. Every veriable that is expandable in the default env See:
https://github.com/rebar/rebar/issues/348 default env:
https://github.com/rebar/rebar/blob/master/src/rebar_port_compiler.erl#L572
As a workaround, set `ERL_EI_LIBDIR` to the actual Erlang's lib
sysroot, so a patches `do_compile` could look like this::

  ERL_CFLAGS = "-I${STAGING_LIBDIR}/erlang/usr/include"
  ERL_LDFLAGS = "-L${STAGING_LIBDIR}/erlang/usr/lib -lerl_interface -lei"
  ERL_EI_LIBDIR = "${STAGING_LIBDIR}/erlang/usr/lib"

  do_compile() {

      # rebar is shit and ERL_LDFLAGS cannot be overridden because it's
      # expandable, see https://github.com/rebar/rebar/issues/348 try to
      # workaround by setting ERL_EI_LIBDIR that is used in the
      # following context:
      # ERL_LDFLAGS=-L$ERL_EI_LIBDIR -lerl_interface -lei'
      oe_runmake ERL_CFLAGS=${ERL_CFLAGS} ERL_EI_LIBDIR=${ERL_EI_LIBDIR} REBAR='rebar -vv' release
  }

Note, `ERL_CLFAGS` must to be set to something meaningful like this::

  ERL_CFLAGS = "-I${STAGING_LIBDIR}/erlang/usr/include"

Otherwise rebar compiled ports will pick up native headers, and you're
left with debugging why integers passed to/from port have
unexplainable values.

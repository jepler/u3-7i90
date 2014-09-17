# u3-7i90

This repository will hold the design files for the level adaptor from ODROID U3 to Mesa 7i90, as well as instructions on building a Linux kernel for the ODROID with the necessary customizations.

# The board

## Theory

The TXB0104 level translator is used for voltage conversion between the ODROID (1.8v I/O standard) and 7i90 (3.3v I/O standard).  I had success running at up to 40MHz, but by the TXB datasheet a safe rate is more like 30MHz.  (in any case, even at 30MHz, setup / cleanup time dominates SPI transaction time so choose a reliable rate rather than a fast one)

## Design Files

The board was designed in Eagle 6.6.  The board files may not be readable in earlier versions.

## Assembling and installing the board

Most of the assembly should be straightforward.  Two things to note:

 * You MUST use appropriate spacers between the two boards to keep the connectors well-aligned.  The 2mm connectors are fragile; I ruined them and had to rework the board, which was not a lot of fun.
 * The connectors go on the top (owl) side, the surface-mount parts go on the bottom (blank) side.
 * I recommend that you work in this order:
  * Solder all items but the connectors
  * Insert the 2mm male headers into the odroid's female headers
  * put the level adapter board on top with the spacers in between
  * solder the 2mm male headers onto the level adapter board
  * solder the 26-pin header

This assembly method is somewhat forgiving of overlong spacers, as long as some of the solder tail of the male headers still sticks into the board.  You want to avoid using too-short spacers, as this will flex the boards.  In theory the assembly distance should be 6mm.

# The software 
## OS
I use the Ubuntu 14.04 image from hardkernel, but probably a debian userspace would work just as well.

## Kernel

You'll have to build your own kernel; there's no packaged kernel at this time.

Use the kernel source tree at https://github.com/jepler/odroid-linux, branch odroid-3.8.13-rt.  The configuration you want is `arch/arm/configs/odroidu_defconfig`.  I build the kernels on an amd64 debian machine using an arm compiler from gcc-4.7-arm-linux-gnueabihf.

TODO: add links to installation instructions; split cross-scripts out 

cross configuring script:

```
ARCH=arm make ${1-menuconfig}
```

cross building script:

```
#!/bin/bash
set -xe

extraversion=

export ARCH=arm
export DEB_HOST_ARCH=armhf

export CONCURRENCY_LEVEL=`getconf _NPROCESSORS_ONLN`
fakeroot make-kpkg --arch arm --cross-compile arm-linux-gnueabihf- --initrd ${extraversion:+--append-to-version=$extraversion} kernel_image kernel_headers

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- ${extraversion:+EXTRAVERSION=$extraversion} zImage
cp arch/arm/boot/zImage ../zImage
```

## LinuxCNC

Use LinuxCNC master branch configured for uspace.  Everything should be just
about normal.

Consider booting with isolcpus=3 to improve latency a bit.

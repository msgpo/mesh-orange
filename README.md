A Small ramdisk system running modern Debian
============================================

This project will create system images for installing a Debian-based
router.

The initial hardwre target is to run on an Orange Pi Zero Single Board
Computer and use the TOP-GS07 RT5572 WiFi adapter for mesh traffic.

In the future, it will create a fully working mesh node, with cjdns
and IPFS software installed.


Building the image
------------------

Multiple board targets are supported, but the all use the same basic
process.  The following commands will build for the Orange Pi Zero

    make -C debian build-depends
    make -C boards/sun8i-h2plus-orangepi-zero build-depends

    make -C boards/sun8i-h2plus-orangepi-zero image

First, any packages required to complete the build are installed -
this needs to be done for both the debian environment and the specific
board environment, which is why there are two lines.

Next, the image is built.  Once the disk image is completed, it will placed
in the `output` dir.

Using the image
---------------

Once the image is built, and you have the disk image file from above.
use "dd" (or similar) to write the image to the raw sdcard.

    lsblk -d -o NAME,SIZE,LABEL
    echo sudo dd if=output/$IMAGE of=$DISK

Warning: dont overwrite the wrong disk!

Booting and using the system
----------------------------

A wireless access point will be automatically started on all detected
wifi adaptors - including those hotplugged after bootup.  Any internet
connection plugged into the ethernet port will be shared out over the
access point.

(Note: There is currently a bug with stopping/restarting the hostapd,
so if you unplug an adaptor, it will not automatically work again until
a reboot is done)

During testing the following default settings are used:

* wifi ssid: `test2`
* wifi passphrase: `bbbbbbbb`
* user: `root`
* pass: `root`

An ssh server is started on bootup, so the simplest way to login is to
connect to the wifi and use the root password.

During development and debugging, it is very helpful to use a serial
console to see the boot messages and login.  It is also might be useful
to read the section below on running this image in an emulator.

NOTE: uncompressing the (currently approximately 120Meg uncompressed) initrd
will take a noticable amount of time.  The network will not be setup until
that has been done, so nothing will happen for a while.  For some reason, this
also applies to the kernel messages.

Tests on one orange pi zero show that the time from power on until a login
prompt on the serial console is about 1 minute.


Test the Debian image using Qemu:
---------------------------------

It is possible to boot the Debian system up in an emulator.  This is
useful during development to speed up testing but is also useful for
exploring the system and learning its features before committing to the
purchase of any hardware.

### Testing with the armhf architecture

The armhf architecture is an environment that is closer to the real-life
Single Board Computers that are expected to be used - and it uses exactly
the same binaries that would be used on those.  However, it is usually
a slower emulator.

    make -C debian build-depends
    make -C boards/qemu_armhf build-depends

    make -C boards/qemu_armhf test

Once the build has completed, it will boot up inside the emulator.  The
console of the emulator is connected to your terminal window.

To exit the emulator, use Ctrl-A then "x"

### Testing with the i386 architecture

While this is not the expected architecture for most physical hardware,
it is a much faster emulation.  Additionally, all the debian packages,
configuration and customisation is the same as the armhf architecture.

    make -C debian build-depends
    make -C boards/qemu_i386 build-depends

    make -C boards/qemu_i386 test

To exit the emulator, use Ctrl-A then "x"

Debian ramdisk builder
----------------------

The debian directory contains the Debian builder - see the README in
that dir for more details.

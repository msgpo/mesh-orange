#
# Build a Orange Pi Zero install
#

DEBIAN_VER = stretch
DEBIAN_ARCH = armhf

# which uboot and device tree is this being built for
UBOOT = linux-u-boot-dev-orangepizero_5.31_armhf
BOARD = sun8i-h2-plus-orangepi-zero
# FIXME - it would be nice if the uboot name was related to the dtb name

SRC_SPL = $(ARMBIAN)/usr/lib/$(UBOOT)/u-boot-sunxi-with-spl.bin
SRC_DTB = ../../linux/build/linux/dtb/$(BOARD).dtb

PART_SIZE_MEGS = 1000

CLEAN_FILES = $(TAG) $(ARMBIAN) $(BUILD)/firmware.cpio $(BUILD)/firmware.lzma $(BOOT) $(BUILD)/mtoolsrc

# Directories
ARMBIAN = $(BUILD)/armbian

BUILD_DEPENDS = \
    multistrap \
    fakeroot \
    u-boot-tools \
    xz-utils \
    mtools \

all: image

include ../common.mk
include ../common-uboot.mk
include ../common-allwinner.mk
include ../common-kernel-local.mk

image: $(DISK_IMAGE)
	mkdir -p ../../output
	cp $< ../../output/$(BOARD).img

test:
	echo this board must be tested by flashing the output
	false

# Everything below this line is HW specific u-Boot startup code

$(ARMBIAN): $(TAG)/armbian
$(TAG)/armbian: armbian.multistrap
	mkdir -p $(ARMBIAN)
	fakeroot /usr/sbin/multistrap -d $(ARMBIAN) -f $<
	$(call tag,armbian)

# Add the kernel specific binaries to this cpio file
$(BUILD)/firmware.cpio: $(TAG)/armbian
	( \
            cd $(ARMBIAN); \
            find lib/firmware -print0 | cpio -0 -H newc -R 0:0 -o \
	) > $@

$(BOOT): $(TAG)/boot
$(TAG)/boot:
	mkdir -p $(BOOT)
	$(call tag,boot)

$(BOOT)/.next: $(TAG)/boot
	touch $@

# TODO
# - rewrite or cleanup these two files for our specific use cases

$(BOOT)/boot.scr: $(TAG)/boot
$(BOOT)/boot.scr: armbian/lib/config/bootscripts/boot-sunxi.cmd
	mkimage -A arm -T script -C none -d $< $@

$(BOOT)/armbianEnv.txt: $(TAG)/boot
$(BOOT)/armbianEnv.txt: armbian/lib/config/bootenv/sunxi-default.txt
	cp $< $@

$(BOOT)/zImage: $(LOCAL_KERNEL)
	cp $< $@
	cp $(dir $<)/.config $(dir $@)

# Combine the various modules to make one big cpio file
$(BUILD)/combined.initrd: $(DEBIAN).lzma $(LOCAL_MODULES) $(BUILD)/firmware.lzma
	cat $^ >$@

$(BOOT)/uInitrd: $(TAG)/boot
$(BOOT)/uInitrd: $(BUILD)/combined.initrd
	mkimage -C lzma -A arm -T ramdisk -d $< $@

BOOT_FILES = \
    $(BOOT)/boot.scr $(BOOT)/armbianEnv.txt \
    $(BOOT)/.next \
    $(BOOT)/zImage \
    $(BOOT)/uInitrd \

BOOT_DTB_FILES = \
    $(BOOT)/dtb/$(BOARD).dtb \

# FIXME - add these to the linux build
BOOT_DTBO_FILES = \
    $(BOOT)/dtb/overlay/sun8i-h3-usbhost0.dtbo \
    $(BOOT)/dtb/overlay/sun8i-h3-usbhost2.dtbo \
    $(BOOT)/dtb/overlay/sun8i-h3-usbhost3.dtbo \

boot: $(BOOT_FILES) $(BOOT_DTB_FILES)

# Everything below this line is packing the built boot dir into a disk image

$(SRC_SPL): $(TAG)/armbian

$(BUILD)/mtoolsrc: Makefile
	echo 'drive z: file="$(DISK_IMAGE).tmp" cylinders=$(PART_SIZE_MEGS) heads=64 sectors=32 partition=1 mformat_only' >$@

$(DISK_IMAGE): $(SRC_SPL) $(BUILD)/mtoolsrc $(BOOT_FILES) $(BOOT_DTB_FILES)
	$(call allwinner_spl,$(SRC_SPL),$@.tmp)
	truncate --size=1025K $@.tmp    # ensure the FAT bootblock is mapped
	$(call uboot_bootdir,$(BUILD)/mtoolsrc,z:,$$((0x100000/512)),$(BOOT_FILES),$(BOOT_DTB_FILES))
	mv $@.tmp $@

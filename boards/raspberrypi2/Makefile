#
# Build a raspberry pi install
#

DEBIAN_VER = stretch
DEBIAN_ARCH = armhf

BOARD = raspberrypi2

# raspberrypi3 uses the same image but making a copy to avoid confusion
BOARD_ALIAS = raspberrypi3

RASPBIAN_BOOT_PKG = raspberrypi-bootloader
RASPBIAN_KERNEL_PKG = raspberrypi-kernel

INITRD_PARTS += $(BUILD)/modules.lzma

CLEAN_FILES += $(TAG)

# Directories
RASPBERRYPI = $(BUILD)/raspberrypi

BUILD_DEPENDS = \
    xz-utils \
    mtools \

TOP_DIR ?= $(abspath ../..)
export TOP_DIR

all: image

include $(TOP_DIR)/mk/common.mk
include $(TOP_DIR)/boards/common-partitions.mk
include $(TOP_DIR)/boards/common-firmware-raspbian.mk
include $(TOP_DIR)/boards/common-cjdns-hack.mk
include $(TOP_DIR)/boards/common-initrd.mk

# Add the alias to the normal image target
image: $(TOP_DIR)/output/$(BOARD_ALIAS).img

$(TOP_DIR)/output/$(BOARD_ALIAS).img: $(DISK_IMAGE)
	@mkdir -p $(dir $@)
	cp $< $@

test:
	echo this board must be tested by flashing the output
	false

$(TAG)/raspberrypi:
	$(TOP_DIR)/scripts/get_deb $(DEBIAN_ARCH) raspbian $(RASPBERRYPI) $(RASPBIAN_BOOT_PKG) $(RASPBIAN_KERNEL_PKG)
	$(call tag,raspberrypi)

# Add the kernel specific binaries to this cpio file
$(BUILD)/modules.cpio: $(TAG)/raspberrypi
	( \
            cd $(RASPBERRYPI); \
            find lib/modules -print0 | cpio -0 -H newc -R 0:0 -o \
	) > $@
CLEAN_FILES += $(BUILD)/modules.cpio

$(BUILD): $(TAG)/build
$(TAG)/build:
	mkdir -p $(BUILD)
	$(call tag,build)

$(BOOT): $(TAG)/boot
$(TAG)/boot:
	mkdir -p $(BOOT)
	$(call tag,boot)

$(BOOT)/initrd: $(TAG)/boot
$(BOOT)/initrd: $(BUILD)/combined.initrd
	cp $< $@

$(BOOT)/cmdline.txt: $(TAG)/boot
$(BOOT)/cmdline.txt: cmdline.txt
	cp $< $@

$(BOOT)/config.txt: $(TAG)/boot
$(BOOT)/config.txt: config.txt
	cp $< $@

# TODO
# - add and enable the pi3-miniuart-bt and pi3-disable-bt overlay files

$(BOOT)/%: $(RASPBERRYPI)/boot/% $(TAG)/boot
	cp $< $@

# What an annoying array of dependancies
# TODO - make this more automatic

$(RASPBERRYPI)/boot/bootcode.bin: $(TAG)/raspberrypi
$(RASPBERRYPI)/boot/fixup_cd.dat: $(TAG)/raspberrypi
$(RASPBERRYPI)/boot/start_cd.elf: $(TAG)/raspberrypi
$(RASPBERRYPI)/boot/kernel7.img: $(TAG)/raspberrypi
$(RASPBERRYPI)/boot/COPYING.linux: $(TAG)/raspberrypi
$(RASPBERRYPI)/boot/LICENCE.broadcom: $(TAG)/raspberrypi
$(RASPBERRYPI)/boot/bcm2709-rpi-2-b.dtb: $(TAG)/raspberrypi
$(RASPBERRYPI)/boot/bcm2710-rpi-3-b.dtb: $(TAG)/raspberrypi
$(RASPBERRYPI)/boot/bcm2710-rpi-3-b-plus.dtb: $(TAG)/raspberrypi
$(RASPBERRYPI)/boot/bcm2710-rpi-cm3.dtb: $(TAG)/raspberrypi

BOOT_FILES = \
    $(BOOT)/bootcode.bin \
    $(BOOT)/start_cd.elf \
    $(BOOT)/fixup_cd.dat \
    $(BOOT)/kernel7.img \
    $(BOOT)/initrd \
    $(BOOT)/cmdline.txt \
    $(BOOT)/config.txt \
    $(BOOT)/COPYING.linux \
    $(BOOT)/LICENCE.broadcom \
    $(BOOT)/bcm2709-rpi-2-b.dtb \
    $(BOOT)/bcm2710-rpi-3-b.dtb \
    $(BOOT)/bcm2710-rpi-3-b-plus.dtb \
    $(BOOT)/bcm2710-rpi-cm3.dtb \

boot: $(BOOT_FILES)

# Everything below this line is packing the built boot dir into a disk image

$(DISK_IMAGE): $(BUILD)/mtoolsrc $(BOOT_FILES)
	$(call image_normal)
	MTOOLSRC=$(BUILD)/mtoolsrc mcopy $(BOOT_FILES) z:
	mv $@.tmp $@

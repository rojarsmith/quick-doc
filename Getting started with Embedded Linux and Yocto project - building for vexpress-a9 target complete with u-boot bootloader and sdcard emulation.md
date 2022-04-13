# Getting started with Embedded Linux and Yocto project - building for vexpress-a9 target complete with u-boot bootloader and sdcard emulation

2021

## Getting started

|  Codename  | Yocto Project Version | Release Date |    Current Version     |                      Support Level                       | Poky Version | BitBake branch |
| :--------: | :-------------------: | :----------: | :--------------------: | :------------------------------------------------------: | :----------: | :------------: |
| Kirkstone  |          3.5          |  April 2022  |                        |       Future - Long Term Support (until Apr. 2024)       |     27.0     |                |
|  Honister  |          3.4          | October 2021 | 3.4.2 (February 2022)  |          Support for 7 months (until May 2022)           |     26.0     |      1.52      |
| Hardknott  |          3.3          |  April 2021  |    3.3.4 (Nov 2021)    | Stable - Support for 11 (was 7) months (until Mar. 2022) |     25.0     |      1.50      |
| Gatesgarth |          3.2          |   Oct 2020   |    3.2.4 (May 2021)    |                           EOL                            |     24.0     |      1.48      |
|  Dunfell   |          3.1          |  April 2020  | 3.1.14 (February 2022) |           Long Term Support (until Apr. 2024)            |     23.0     |      1.46      |



- `build-appliance-image`: An example virtual machine that contains all the pieces required to run builds using the build system as well as the build system itself. You can boot and run the image using either the [VMware Player](https://www.vmware.com/products/player/overview.html) or [VMware Workstation](https://www.vmware.com/products/workstation/overview.html). For more information on this image, see the [Build Appliance](https://www.yoctoproject.org/software-item/build-appliance) page on the Yocto Project website.
- `core-image-base`: A console-only image that fully supports the target device hardware.
- `core-image-full-cmdline`: A console-only image with more full-featured Linux system functionality installed.
- `core-image-lsb`: An image that conforms to the Linux Standard Base (LSB) specification. This image requires a distribution configuration that enables LSB compliance (e.g. `poky-lsb`). If you build `core-image-lsb` without that configuration, the image will not be LSB-compliant.
- `core-image-lsb-dev`: A `core-image-lsb` image that is suitable for development work using the host. The image includes headers and libraries you can use in a host development environment. This image requires a distribution configuration that enables LSB compliance (e.g. `poky-lsb`). If you build `core-image-lsb-dev` without that configuration, the image will not be LSB-compliant.
- `core-image-lsb-sdk`: A `core-image-lsb` that includes everything in the cross-toolchain but also includes development headers and libraries to form a complete standalone SDK. This image requires a distribution configuration that enables LSB compliance (e.g. `poky-lsb`). If you build `core-image-lsb-sdk` without that configuration, the image will not be LSB-compliant. This image is suitable for development using the target.
- `core-image-minimal`: A small image just capable of allowing a device to boot.
- `core-image-minimal-dev`: A `core-image-minimal` image suitable for development work using the host. The image includes headers and libraries you can use in a host development environment.
- `core-image-minimal-initramfs`: A `core-image-minimal` image that has the Minimal RAM-based Initial Root Filesystem (initramfs) as part of the kernel, which allows the system to find the first “init” program more efficiently. See the [PACKAGE_INSTALL](https://docs.yoctoproject.org/ref-manual/variables.html#term-PACKAGE_INSTALL) variable for additional information helpful when working with initramfs images.
- `core-image-minimal-mtdutils`: A `core-image-minimal` image that has support for the Minimal MTD Utilities, which let the user interact with the MTD subsystem in the kernel to perform operations on flash devices.
- `core-image-rt`: A `core-image-minimal` image plus a real-time test suite and tools appropriate for real-time use.
- `core-image-rt-sdk`: A `core-image-rt` image that includes everything in the cross-toolchain. The image also includes development headers and libraries to form a complete stand-alone SDK and is suitable for development using the target.
- `core-image-sato`: An image with Sato support, a mobile environment and visual style that works well with mobile devices. The image supports X11 with a Sato theme and applications such as a terminal, editor, file manager, media player, and so forth.
- `core-image-sato-dev`: A `core-image-sato` image suitable for development using the host. The image includes libraries needed to build applications on the device itself, testing and profiling tools, and debug symbols. This image was formerly `core-image-sdk`.
- `core-image-sato-sdk`: A `core-image-sato` image that includes everything in the cross-toolchain. The image also includes development headers and libraries to form a complete standalone SDK and is suitable for development using the target.
- `core-image-testmaster`: A “controller” image designed to be used for automated runtime testing. Provides a “known good” image that is deployed to a separate partition so that you can boot into it and use it to deploy a second image to be tested. You can find more information about runtime testing in the “[Performing Automated Runtime Testing](https://docs.yoctoproject.org/dev-manual/common-tasks.html#performing-automated-runtime-testing)” section in the Yocto Project Development Tasks Manual.
- `core-image-testmaster-initramfs`: A RAM-based Initial Root Filesystem (initramfs) image tailored for use with the `core-image-testmaster` image.
- `core-image-weston`: A very basic Wayland image with a terminal. This image provides the Wayland protocol libraries and the reference Weston compositor. For more information, see the “[Using Wayland and Weston](https://docs.yoctoproject.org/dev-manual/common-tasks.html#using-wayland-and-weston)” section in the Yocto Project Development Tasks Manual.
- `core-image-x11`: A very basic X11 image with a terminal.



### Installing required packages to run yocto

```bash
# Installing required packages to run yocto
sudo apt install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev

# >= 3.4
sudo apt install zstd
```

### Downloading Poky

```bash
git clone git://git.yoctoproject.org/poky

cd poky

git branch -a

# LTS 2020
git checkout tags/yocto-3.1.4 -b yocto-3.1.4-local

git checkout tags/yocto-3.1.14 -b yocto-3.1.14-local

# or Stable 2021, kernel >= 5.14, not work (?)
git checkout tags/yocto-3.3.4 -b yocto-3.3.4-local

git checkout tags/yocto-3.4.2 -b yocto-3.4.2-local

git branch
```

## Building sample image for virt target

```bash
# Initialize the Build Environment
source oe-init-build-env build-virt

vi conf/local.conf

MACHINE ?= "qemuarm"

# build process took about N hours
bitbake core-image-minimal

# output images in dir:
# /build-virt/tmp/deploy/images/qemuarm/

# Run the image on qemu
runqemu qemuarm nographic
```

## Building for vexpress-a9

```bash
# In poky directory clone the layer
git clone https://github.com/OverC/meta-qemuarma9.git

# Run again if `bitbake: command not found`
source oe-init-build-env build-vexpressa9

# If not work
vi ../meta-qemuarma9/conf/layer.conf

LAYERSERIES_COMPAT_qemuarma9 = "dunfell gatesgarth hardknott honister kirkstone"

# >= 3.4
vi ../meta-qemuarma9/conf/machine

require conf/machine/include/arm/armv7a/tune-cortexa9.inc

bitbake-layers add-layer ../meta-qemuarma9/

cat conf/bblayers.conf

BBLAYERS ?= " \
  /hdd/poky/meta \
  /hdd/poky/meta-poky \
  /hdd/poky/meta-yocto-bsp \
  /hdd/poky/meta-qemuarma9 \
"

vi conf/local.conf

MACHINE ?= "qemuarma9"

/meta-qemuarma9 ...pes-kernel/linux/linux-yocto-dev.bbappend → recipes-kernel/linux/linux-yocto_%.bbappend
 
# v5.14/standard/arm-versatile-926ejs
 
# WR qemuarma9 configuration, not supported in Yocto
KBRANCH_qemuarma9 ?= "v5.4/standard/arm-versatile-926ejs"
KMACHINE_qemuarma9 ?= "qemuarma9"
KERNEL_DEVICETREE_qemuarma9 = "vexpress-v2p-ca9.dtb"
COMPATIBLE_MACHINE_qemuarma9 = "qemuarma9"



# Delete linux-yocto_5.2.bbappend and linux-yocto_4.8.bbappend

# If any error because of fetching, try again & again
bitbake core-image-minimal

# Resize for SD Card
qemu-img resize tmp/deploy/images/qemuarma9/core-image-minimal-qemuarma9-20220222090323.rootfs.ext4 4G

qemu-img resize tmp/deploy/images/qemuarma9/core-image-minimal-qemuarma9-20220226121058.rootfs.ext4 4G

# Reach a `end Kernel panic` of course since we didn't attach a root filesystem.
qemu-system-arm -machine vexpress-a9 -m 256 -kernel tmp/deploy/images/qemuarma9/zImage -dtb tmp/deploy/images/qemuarma9/vexpress-v2p-ca9.dtb -append "console=ttyAMA0" -nographic
```

## Building u-boot

```bash
vi ../meta-qemuarma9/conf/machine/qemuarma9.conf

#Include u-boot
EXTRA_IMAGEDEPENDS += "u-boot"
UBOOT_MACHINE = "vexpress_ca9x4_defconfig"

# Include u-boot elf file in the images folder
UBOOT_ELF = "u-boot"

bitbake core-image-minimal

# run qemu with -kernel u-boot.elf:

qemu-system-arm -machine vexpress-a9 -m 256 -kernel tmp/deploy/images/qemuarma9/u-boot.elf -nographic

bdinfo

arch_number = 0x000008e0
boot_params = 0x60002000
# DRAM bank 0 addresses, this is what we will use to load kernel and device tree.
DRAM bank   = 0x00000000
-> start    = 0x60000000
-> size     = 0x10000000

Exit: Ctrl+A, X
```

### Prepare SD Card image

```bash
pushd ../meta-qemuarma9/

mkdir wic
touch wic/qemuarma9-yocto.wks
vi wic/qemuarma9-yocto.wks

part /boot --source bootimg-partition --ondisk mmcblk0 --fstype=ext4 --label boot --align 2 --use-uuid
part / --source rootfs --ondisk mmcblk0 --fstype=ext4 --label root --align 2 --use-uuid

# 3rd
part /mnt --fixed-size 100M --ondisk mmcblk0 --fstype=ext4 --label storage --align 131072 --use-uuid

# /mnt/p3 will not mount because dictionary not exit.
# Can mount manually after make dictionary.
# mount -t ext4 /dev/mmcblk0p3 p3

# ???
part /p3 --size 100M --ondisk mmcblk0 --fstype=ext4 --label storage --align 1024 --use-uuid
part /mnt --ondisk mmcblk --fstype=ext4 --label rootfs2 --align 4096 --fixed-size=800M
part /opt --source rootfs --rootfs-dir=${IMAGE_ROOTFS}/opt --ondisk sda --fstype=ext4 --label opt --align 8192

vi conf/machine/qemuarma9.conf

# Wic configuration
IMAGE_FSTYPES += "wic"
WKS_FILE ?= "qemuarma9-yocto.wks"

# Files that will be included in boot partition
IMAGE_BOOT_FILES ?= "zImage vexpress-v2p-ca9.dtb"
# for OTA
IMAGE_BOOT_FILES ?= "zImage;kernel_a zImage;kernel_b vexpress-v2p-ca9.dtb;dtb_a vexpress-v2p-ca9.dtb;dtb_b uboot.env"

# Remove unneeded configurations from the original bsp:

# Set image size for MMC disk
#IMAGE_ROOTFS_EXTRA_SPACE = "0"
#IMAGE_ROOTFS_SIZE = "3801088"
#INITRAMFS_MAXSIZE = "3801088"
#IMAGE_ROOTFS_ALIGNMENT = "1"

# Include u-boot elf file in the images folder
UBOOT_ELF = "u-boot"
UBOOT_ENTRYPOINT = "0x60100000"
UBOOT_LOADADDRESS = "0x60100000"

vi conf/machine/qemuarma9.inc

#ROOT_FLASH_SIZE = "280"

popd

bitbake core-image-minimal

fdisk -l tmp/deploy/images/qemuarma9/core-image-minimal-qemuarma9.wic
```

Run this in the output images directory. You will now have the file sd.img ready to be used as the emulated SD Card.

```bash
pushd tmp/deploy/images/qemuarma9
touch mksd.sh
vi mksd.sh

#!/bin/sh

file="sd.img"
		
if [ -f "$file" ] ; then
    read -p "An sd.img file already exists. Are you sure you want to delete it and create a new one? (y/n) " input
    if [ "$input" != "y" ]; then
        exit
    fi

    echo "Removing old sd.img"
    rm "$file"
fi
		
echo "Creating empty sd.img file"
dd if=/dev/zero of=sd.img bs=1M count=1000

echo "Copying content from wic file"
dd if=core-image-minimal-qemuarma9.wic of=sd.img conv=notrunc
echo "Resize rootfs partition to take the whole sd.img size"
parted -s sd.img resizepart 2 100% # Not work with > 2 part

chmod 777 mksd.sh
./mksd.sh

# Resize for SD Card manually, for part > 2
qemu-img resize sd.img 1G

fdisk -l sd.img

popd
```

### Booting Linux from SD Card

```bash
qemu-system-arm -machine vexpress-a9 -m 256 -kernel tmp/deploy/images/qemuarma9/zImage -dtb tmp/deploy/images/qemuarma9/vexpress-v2p-ca9.dtb -nographic -append "console=ttyAMA0 root=/dev/mmcblk0" -sd tmp/deploy/images/qemuarma9/core-image-minimal-qemuarma9.ext4

qemu-system-arm -machine vexpress-a9 -m 256 -kernel tmp/deploy/images/qemuarma9/u-boot.elf -nographic -sd tmp/deploy/images/qemuarma9/sd.img

# uboot to load the image
ext4load mmc 0:1 0x60100000 zImage
ext4load mmc 0:1 0x62000000 vexpress-v2p-ca9.dtb
setenv bootargs console=ttyAMA0 root=/dev/mmcblk0p2
bootz 0x60100000 - 0x62000000
```

### Editing u-boot configuration to correctly autoboot

pushd ../meta-qemuarma9/

We will create the following folder/files inside meta-qemuarma9 folder:

```
├── recipes-bsp
│   └── u-boot
│       ├── files
│       │   └── fragment.cfg
│       └── u-boot_%.bbappend
```

Edit fragment.cfg and add the following lines:

```
CONFIG_USE_BOOTARGS=y
CONFIG_BOOTARGS="console=ttyAMA0 root=/dev/mmcblk0p2"
CONFIG_BOOTCOMMAND="ext4load mmc 0:1 0x60100000 zImage\; ext4load mmc 0:1 0x62000000 vexpress-v2p-ca9.dtb\; bootz 0x60100000 - 0x62000000"
```

Edit u-boot_%.bbappend and add these lines:

```
FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
SRC_URI += "file://fragment.cfg"
```

popd

bitbake core-image-minimal

```bash
qemu-system-arm -machine vexpress-a9 -m 256 -kernel tmp/deploy/images/qemuarma9/u-boot.elf -nographic -sd tmp/deploy/images/qemuarma9/sd.img
```



### Adding packages to our distro

```bash
# poky directory
git clone http://cgit.openembedded.org/meta-openembedded
cd meta-openembedded
git checkout dunfell
cd ..
source oe-init-build-env <build-env-name>
bitbake-layers add-layer ../meta-openembedded/meta-oe
```



Error “ParseError at /home/dev/service/poky/meta-openembedded/meta-oe/recipes-dbs/postgresql/postgresql.inc:39: Could not inherit file classes/python3targetconfig.bbclass”



vi poky/meta/classes/python3targetconfig.bbclass

```bash
inherit python3native

EXTRA_PYTHON_DEPENDS ?= ""
EXTRA_PYTHON_DEPENDS_class-target = "python3"
DEPENDS_append = " ${EXTRA_PYTHON_DEPENDS}"

do_configure_prepend_class-target() {
        export _PYTHON_SYSCONFIGDATA_NAME="_sysconfigdata"
}

do_compile_prepend_class-target() {
        export _PYTHON_SYSCONFIGDATA_NAME="_sysconfigdata"
}

do_install_prepend_class-target() {
        export _PYTHON_SYSCONFIGDATA_NAME="_sysconfigdata"
}
```

We do this by adding this line into conf/local.conf:

```bash
IMAGE_INSTALL_append = " nano"
```

Now run bitbake and create your sd.img again to add the new package into the sd card.

```bash
bitbake core-image-minimal
pushd tmp/deploy/images/qemuarma9/
./mksd.sh
qemu-img resize sd.img 1G
qemu-system-arm -machine vexpress-a9 -m 256 -kernel u-boot.elf -nographic -sd sd.img
```

## SWUpdate

```bash
# poky directory
git clone https://github.com/sbabic/meta-swupdate.git

cd meta-swupdate
git checkout dunfell
cd ..
source oe-init-build-env <build-env-name>
bitbake-layers add-layer ../meta-swupdate

# Disable MTD
# → Swupdate Settings → General Configuration
bitbake -c menuconfig swupdate

bitbake swupdate

bitbake linux-yocto -c menuconfig

# meta-oe is must
cat conf/bblayers.conf


git clone https://github.com/sbabic/meta-swupdate-boards.git -b master
source oe-init-build-env build-vexpressa9
bitbake-layers add-layer ../meta-swupdate-boards


vi conf/local.conf
IMAGE_INSTALL_append = " nano swupdate swupdate-www"

bitbake core-image-minimal

ps | grep swupdate | grep -v grep

  288 root     47860 S    /usr/bin/swupdate -v -w -r /www -p 8080
  309 root     47860 S    /usr/bin/swupdate -v -w -r /www -p 8080


df -h /

fdisk -l

# mount multi partitions
sudo apt install kpartx
sudo kpartx -a tmp/deploy/images/qemuarma9/sd.img sdm
sudo mount -o rw -t ext4 /dev/mapper/loop13p3 sdm/
sudo umount /dev/mapper/loop13p3

In meta-qemuarma9/conf/machine/qemuarm9.conf we will update IMAGE_BOOT_FILES variable to this:

IMAGE_BOOT_FILES ?= "zImage;kernel_a zImage;kernel_b vexpress-v2p-ca9.dtb;dtb_a vexpress-v2p-ca9.dtb;dtb_b uboot.env"


part /boot --source bootimg-partition --ondisk mmcblk0 --fstype=ext4 --label boot --align 1024 --use-uuid --extra-space 50 --mkfs-extraopts="-O ^metadata_csum"
		
part --source rootfs --fstype=ext4 --ondisk mmcblk0 --use-uuid --label root_a --align 1024 --extra-space 300
		
part --source rootfs --fstype=ext4 --ondisk mmcblk0 --use-uuid --label root_b --align 1024 --extra-space 300



part /boot --source bootimg-partition --ondisk mmcblk0 --fstype=ext4 --label boot --align 1024 --use-uuid --extra-space 5 --mkfs-extraopts="-O ^metadata_csum"

part / --source rootfs --ondisk mmcblk0 --fstype=ext4 --label root_a --align 1024 --use-uuid --extra-space 3

part / --source rootfs --ondisk mmcblk0 --fstype=ext4 --label root_b --align 1024 --use-uuid --extra-space 3

part /mnt --fixed-size 100M --ondisk mmcblk0 --fstype=ext4 --label storage --align 2 --use-uuid



Now the output wic file will contain the needed partition scheme.

touch sw-description

mmcblk0          tty              tty38            ttyAMA1
mmcblk0p1        tty0             tty39            ttyAMA2
mmcblk0p2        tty1             tty4             ttyAMA3
mmcblk0p3        tty10            tty40            urandom
mmcblk0p4

blkid -s LABEL -o value /dev/mmcblk0p3

fw_env.config

# MTD device name       Device offset   Env. size       Flash sector size
/dev/mtd0               0xc0000         0x2000          0x10000


dmesg

touch gen-swupdate-image.sh


#!/bin/sh

BOARD_DIR=$(dirname $0)

echo $BOARD_DIR
echo $BINARIES_DIR

IMG_FILES="sw-description rootfs.tar.bz2"

for f in ${IMG_FILES} ; do
	echo ${f}
done | cpio -ovL -H crc > update.swu


./gen-swupdate-image.sh
cat update.swu | cpio -it

sgdisk -A 4:toggle:2 -A 5:toggle:2 /dev/mmcblk0

swupdate -i /mnt/update.swu -e rootfs,rootfs-2

[ERROR] : SWUPDATE failed [0] ERROR : Configuration file /etc/fw_env.config wrong or corrupted

bitbake virtual/kernel -c menuconfig

> Device Drivers > Memory Technology Device (MTD) support
Enable UBI - Unsorted block images  --->


IMAGE_INSTALL_append = " nano swupdate swupdate-www swupdate-progress swupdate-client json-c"

```


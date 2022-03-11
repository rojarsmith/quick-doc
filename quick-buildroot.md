# Quick Buildroot

## Command

```bash
make linux-menuconfig
```

## qemu_arm_vexpress

```bash
sudo apt install gcc build-essential bison flex gettext tcl sharutils libncurses-dev zlib1g-dev exuberant-ctags g++ texinfo patch vim libtool bc git

sudo apt-get install qemu-system-arm

qemu-system-arm --version

wget https://buildroot.org/downloads/buildroot-2021.02.9.tar.gz
tar zxvf buildroot-*

make list-defconfigs

# config for qemu_arm_vexpress
make qemu_arm_vexpress_defconfig
make qemu_arm_vexpress_tz_defconfig

make menuconfig

make -j 4

# root login

board/qemu/post-image.sh

output/images/start-qemu.sh

qemu-system-arm -M vexpress-a9 -smp 1 -m 256 -kernel output/images/zImage -dtb output/images/vexpress-v2p-ca9.dtb -drive file=output/images/rootfs.ext2,if=sd,format=raw -append "console=ttyAMA0,115200 root=/dev/mmcblk0" -serial stdio -net nic,model=lan9118 -net user

qemu-system-arm -M vexpress-a9 -smp 4 -m 1024M -kernel output/images/zImage -append "root=/dev/mmcblk0 console=ttyAMA0 loglevel=8" -dtb output/images/vexpress-v2p-ca9.dtb -sd output/images/rootfs.ext2 -nographic

/media/dev/7D6A-1A1D

vi board/qemu/arm-vexpress-tz/sw-description


software = {
	version = "0.1.0";
	rootfs = {
		rootfs-1: {
			images: (
			{
				filename = "rootfs.ext4.gz";
				compressed = true;
				device = "/dev/mmcblk0p4";
			});
		}
		rootfs-2: {
			images: (
			{
				filename = "rootfs.ext4.gz";
				compressed = true;
				device = "/dev/mmcblk0p5";
			});
		}
	}
}


vi board/qemu/arm-vexpress-tz/gen-swupdate-image.sh

#!/bin/bash

BOARD_DIR=$(dirname $0)
BINARIES_DIR=output/images

echo ${BOARD_DIR}/sw-description
echo ${BINARIES_DIR}

cp ${BOARD_DIR}/sw-description ${BINARIES_DIR}

IMG_FILES="sw-description rootfs.ext2.gz"

pushd ${BINARIES_DIR}
for f in ${IMG_FILES} ; do
	echo ${f}
done | cpio -ovL -H crc > buildroot.swu
popd


# In Filesystem images, enable the gzip compression method for the ext2/3/4 root filesystem, so that a rootfs.ext4.gz image is generated.

cat output/images/buildroot.swu | cpio -it

# 250MB
dd if=/dev/zero of=uboot.disk bs=1M count=250
sgdisk -n 0:0:+10M -c 0:kernel uboot.disk
sgdisk -n 0:0:0 -c 0:rootfs uboot.disk

sgdisk -p uboot.disk

LOOPDEV=`sudo losetup -f`
echo $LOOPDEV
sudo losetup $LOOPDEV uboot.disk
sudo partprobe $LOOPDEV
sudo losetup -l
ls /dev/loop*

sudo losetup -d /dev/loop13

sudo mkfs.ext2 /dev/loop13p1
sudo mkfs.ext2 /dev/loop13p2
mkdir p1 p2
sudo mount -t ext2 /dev/loop13p1 p1
sudo mount -t ext2 /dev/loop13p2 p2
df -h

sudo cp buildroot-2021.02.8/output/images/zImage /home/dev/service/p1

sudo cp buildroot-2021.02.8/output/images/vexpress-v2p-ca15_a7.dtb /home/dev/service/p1
sudo cp buildroot-2021.02.8/output/images/vexpress-v2p-ca9.dtb /home/dev/service/p1
ls /home/dev/service/p1

# Filesystem images
# tar the root filesystem
sudo tar -C /destination/of/extraction -xf images/rootfs.tar
sudo tar -C buildroot-2021.02.8/output/images/rootfs -xf buildroot-2021.02.8/output/images/rootfs.tar

sudo cp buildroot-2021.02.8/output/images/rootfs/* /home/dev/service/p2 -arf
ls /home/dev/service/p2

sudo umount p1 p2
sudo losetup -d /dev/loop13

qemu-system-arm -M vexpress-a9 -m 1024M -smp 1 -nographic -kernel u-boot.bin -sd ./uboot.disk

qemu-system-arm -M vexpress-a9 -m 1024M -smp 1 -nographic -kernel buildroot-2021.02.8/output/images/u-boot.bin -sd ./uboot.disk

qemu-system-arm  -machine virt -machine secure=on -cpu cortex-a15  -smp 1 -s -m 1024 -d unimp    -netdev user,id=vmnic -device virtio-net-device,netdev=vmnic  -semihosting-config enable,target=native  -bios bl1.bin  ${EXTRA_ARGS}


qemu-system-arm -machine help
```

## Build root+arm+qemu

```bash
sudo apt install build-essential bison flex gcc-arm-linux-gnueabihf libncurses-dev libssl-dev

sudo apt install qemu-system-arm

wget https://buildroot.org/downloads/buildroot-2021.02.9.tar.gz
tar zxvf buildroot-*
cd buildroot-2021.02.9/

# List support board
qemu-system-arm -machine help

# Choose vexpress-a9

git clone https://source.denx.de/u-boot/u-boot.git

# Exists config/vexpress_ca9x4_defconfig

cd u-boot

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- vexpress_ca9x4_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4


make qemu_arm_vexpress_defconfig

make menuconfig

# Add Target package=>System tools=>SWUpdate

# Change build options=>configs name

# which arm-linux-gnueabihf-gcc
# /usr/bin/arm-linux-gnueabihf-gcc
# $(ARCH)-linux-gnueabihf

ls /usr/arm-linux-gnueabihf/include
cat /usr/arm-linux-gnueabihf/include/linux/version.h 
# Wath Hx16 328,782 => 05 04 4E => 5.4.78

# ttyAMA0
# lz4

# /usr
# 

make savedefconfig

make -j4

git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git kernel
git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
cd linux

cp ../buildroot-2021.02.9/output/images/rootfs.cpio.lz4 .

make ARCH=arm vexpress_defconfig
make ARCH=arm menuconfig

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4


dd if=/dev/zero of=sd.img bs=4096 count=4096
mkfs.vfat sd.img
sudo mount sd.img /mnt/ -o loop,rw
sudo cp ../buildroot-2021.02.9/output/images/zImage /mnt/
sudo cp ../kernel/arch/arm/boot/zImage /mnt/
sudo cp ../buildroot-2021.02.9/output/images/vexpress-v2p-ca9.dtb /mnt/
sudo cp ../kernel/arch/arm/boot/dts/vexpress-v2p-ca9.dtb /mnt/
sudo umount /mnt


qemu-system-arm -nographic -M vexpress-a9 -m 512M -kernel ../u-boot/u-boot -sd sd.img


cd u-boot
qemu-system-arm -M vexpress-a9 -m 256 -kernel u-boot -nographic

fatload mmc 0:0 0x62008000 zImage
fatload mmc 0:0 0x64008000 vexpress-v2p-ca9.dtb

bootz 0x62008000 - 0x64008000
```

## SWUpdate

```bash
sudo apt install build-essential libncurses-dev

make qemu_x86_defconfig

# Add Target package=>System tools=>SWUpdate

make -j4
```

## THIS_IS_NOT_YOUR_ROOT_FILESYSTEM

```bash
"Filesystem images"

"tar the root filesystem"
```

## This example is based of ARM versatilepb board

```
make CROSS_COMPILE=arm-none-eabi- versatilepb_config
make CROSS_COMPILE=arm-none-eabi- qemu_arm_versatile_defconfig
make CROSS_COMPILE=arm-none-eabi- all
```

## Creating the Flash image 

download u-boot-xxx.x source tree and extract it * cd into the source tree directory and build it

```
mkimage -A arm -C none -O linux -T kernel -d zImage -a 0x00010000 -e 0x00010000 zImage.uimg
mkimage -A arm -C none -O linux -T ramdisk -d rootfs.img.gz -a 0x00800000 -e 0x00800000 rootfs.uimg
dd if=/dev/zero of=flash.bin bs=1 count=6M
dd if=u-boot.bin of=flash.bin conv=notrunc bs=1
dd if=zImage.uimg of=flash.bin conv=notrunc bs=1 seek=2M
dd if=rootfs.uimg of=flash.bin conv=notrunc bs=1 seek=4M
```

**Booting Linux**

To boot Linux we can finally call:

```
qemu-system-arm -M versatilepb -m 128M -kernel flash.bin -serial stdio
```

## Virtual USB Volume

To create a virtual USB volume use the following steps:

Use [`fallocate`](http://man7.org/linux/man-pages/man2/fallocate.2.html) to create a new file:

```bash
fallocate -l 256M vud1.img
```

Format it:

```bash
mkfs -t ext4 vud1.img
# mkfs -t ext2 vud1.img
```

Create a mount point:

```bash
sudo mkdir /media/usb_mount_point
```

mount it:

```bash
sudo mount -t auto -o loop virtual_usb.img /media/usb_mount_point
```

umount :

```bash
sudo umount /media/usb_mount_point
```

## Virtual Usb Disk

```bash
qemu-img create -f raw vud1.img 256M # virtual usb disk

ls -l vud1.img

mkfs.vfat vud1.img

qemu-system-arm <other options ...> \
	-drive if=none,format=raw,id=disk1,file=vud1.img \
	-device ich9-usb-ehci1,id=usb \
	-device usb-storage,bus=usb.0,drive=disk1
	
	
qemu-system-arm -M vexpress-a9 -kernel output/build/uboot-2021.01/u-boot -nographic -usb -drive if=none,format=raw,id=disk1,file=vud1.img -device usb-storage,drive=disk1
```

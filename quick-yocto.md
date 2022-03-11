# Quick  Yocto

## Raspberry Pi 3

```bash
sudo apt install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev

mkdir my-rpi && cd my-rpi
git clone -b hardknott git://git.yoctoproject.org/poky.git
git clone -b hardknott git://git.yoctoproject.org/meta-raspberrypi

source poky/oe-init-build-env build-rpi

bitbake-layers add-layer ../meta-raspberrypi

# Not generate qemu environment
sed -i 's/^MACHINE.*/MACHINE ?= "raspberrypi3"/g' conf/local.conf
# Generate qemu environment
sed -i 's/^MACHINE.*/MACHINE ?= "qemuarm"/g' conf/local.conf

sed -i '/^#DL_DIR ?= "${TOPDIR}\/downloads"/ a DL_DIR ?= \"${HOME}/service/yocto-downloads"' conf/local.conf
sed -i 's/^PACKAGE_CLASSES.*/PACKAGE_CLASSES ?= "package_ipk"/g' conf/local.conf

echo 'RPI_USE_U_BOOT = "1"' >> conf/local.conf
echo 'ENABLE_UART = "1"' >> conf/local.conf

bitbake core-image-minimal

# all files for deploy
ls tmp/deploy/images/raspberrypi3/

bzip -Dk core-image-minimal-raspberrypi3.wic.bz2
sudo dd if=core-image-minimal-raspberrypi3.wic of=${SD_CARD} bs=40960

# SWUpdate

git clone -b hardknott http://cgit.openembedded.org/meta-openembedded

bitbake-layers add-layer ../meta-openembedded/meta-oe

git clone -b hardknott https://github.com/sbabic/meta-swupdate.git

bitbake-layers add-layer ../meta-swupdate

git clone -b hardknott https://github.com/sbabic/meta-swupdate-boards.git

bitbake-layers add-layer ../meta-swupdate-boards

bitbake-layers show-recipes

bitbake core-image-minimal


# Qemu
qemu-system-arm -M qemuarm -smp 1 -m 256 -kernel tmp/deploy/images/raspberrypi3/uImage -dtb tmp/deploy/images/raspberrypi3/bcm2710-rpi-3-b-plus.dtb -drive file=tmp/deploy/images/raspberrypi3/core-image-minimal-raspberrypi3.ext3,if=sd,format=raw -append "console=ttyAMA0,115200 root=/dev/mmcblk0" -serial stdio -net nic,model=lan9118 -net user
```

## WIC

.wks

example

```bash
part /opt --source rootfs --rootfs-dir=${IMAGE_ROOTFS}/opt --ondisk sda --fstype=ext4 --label opt --align 8192
part /mnt/p3 --fixed-size 100M --ondisk mmcblk0 --fstype=ext4 --label storage --align 131072 --use-uuid
sudo mount -t ext4 /dev/mmcblk0p3 p3
```

example IMX

```bash
part u-boot --source rawcopy --sourceparams="file=imx-boot" --ondisk sda --no-table --align ${IMX_BOOT_SEEK}
part /boot --source bootimg-partition --ondisk sda --fstype=vfat --label boot --active --align 8192 --size 64
part / --source rootfs --ondisk sda --fstype=ext4 --label root --exclude-path=home/ --exclude-path=opt/ --align 8192
part /home --source rootfs --rootfs-dir=${IMAGE_ROOTFS}/home --ondisk sda --fstype=ext4 --label home --align 8192
part /opt --source rootfs --rootfs-dir=${IMAGE_ROOTFS}/opt --ondisk sda --fstype=ext4 --label opt --align 8192
bootloader --ptable msdos
```


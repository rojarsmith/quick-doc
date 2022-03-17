# Quick  Yocto

## Qemu+RAUC

```bash
git clone https://git.yoctoproject.org/poky

pushd poky

git checkout tags/yocto-3.4.2 -b yocto-3.4.2-local

popd

git clone -b honister https://github.com/rauc/meta-rauc.git

git clone -b master https://github.com/rauc/meta-rauc-community.git

source poky/oe-init-build-env build-qemux86-64-rauc

sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev zstd liblz4-tool

bitbake-layers add-layer ../meta-rauc-community/meta-rauc-qemux86

bitbake-layers add-layer ../meta-rauc

bitbake-layers show-appends

vi conf/local.conf

DISTRO_FEATURES += "rauc"
CORE_IMAGE_EXTRA_INSTALL += "rauc"
MACHINE_FEATURES:append = " pcbios efi"
EXTRA_IMAGEDEPENDS += "ovmf"
PREFERRED_RPROVIDER_virtual-grub-bootconf = "rauc-qemu-grubconf"
INIT_MANAGER = "systemd"
EXTRA_IMAGE_FEATURES += "ssh-server-openssh"
# For hawkbit
IMAGE_INSTALL:append = " rauc-hawkbit-updater"

# Reusing the sstate-cache and downloads.
# Move 2 dirs outside.
DL_DIR ?= "${TOPDIR}/../downloads"
SSTATE_DIR ?= "${TOPDIR}/../sstate-cache"

pushd ../meta-rauc-community/

./create-example-keys.sh

cat ../build-qemux86-64-rauc/conf/site.conf

popd
```

### Building The QEMU System Image

```bash
bitbake core-image-minimal

ls -1 tmp/deploy/images/qemux86-64

runqemu nographic slirp ovmf wic core-image-minimal

# Login: root

# show help
rauc -h

# rootfs.1 boot status: bad is correct.
rauc status

# Build rauc OTA package
bitbake qemu-demo-bundle

ls tmp/deploy/images/qemux86-64/qemu-demo-bundle-qemux86-64.raucb

# check the content of this bundle
bitbake rauc-native -c addto_recipe_sysroot

oe-run-native rauc-native rauc info --keyring="/home/dev/service/app/build-qemux86-64-rauc/example-ca/ca.cert.pem" tmp/deploy/images/qemux86-64/qemu-demo-bundle-qemux86-64.raucb

# specify the verification keyring
eval `grep RAUC_KEYRING_FILE conf/site.conf`

oe-run-native rauc-native rauc info --keyring=$RAUC_KEYRING_FILE tmp/deploy/images/qemux86-64/qemu-demo-bundle-qemux86-64.raucb
```

### RUAC packing

vi ../meta-rauc/recipes-support/rauc-hawkbit-updater/files/do.sh

```bash
remote 1

# or

local 1
```

vi ../meta-rauc/recipes-support/rauc-hawkbit-updater/files/config.conf

```ini
[client]
# host or IP and optional port
hawkbit_server            = bitdove-1.elecdove.com

# true = HTTPS, false = HTTP
ssl                       = true

# validate ssl certificate (only use if ssl is true)
ssl_verify                = false

# Tenant id
tenant_id                 = DEFAULT

# Target name (controller id)
target_name               = dev02

# Security token
auth_token                = 60a78431e1881c2909be6478c3711476

# Or gateway_token can be used instead of auth_token
#gateway_token             = cb115a721af28f781b493fa467819ef5

# Temporay file RAUC bundle should be downloaded to
bundle_download_location  = /tmp/bundle.raucb

# time in seconds to wait before retrying
retry_wait                = 60

# connection timeout in seconds
connect_timeout           = 20

# request timeout in seconds
timeout                   = 60

# time to be below "low_speed_rate" to trigger the low speed abort
low_speed_time            = 0

# average transfer speed to be below during "low_speed_time" seconds
low_speed_rate            = 0

# reboot after a successful update
post_update_reboot        = false

# debug, info, message, critical, error, fatal
log_level                 = message

# Every key / value under [device] is sent to HawkBit (target attributes),
# and can be used in target filter.
[device]
mac_address               = ff:ff:ff:ff:ff:ff
hw_revision               = 2
model                     = T1
```

vi ../meta-rauc/recipes-support/rauc-hawkbit-updater/rauc-hawkbit-updater_%.bbappend

```bash
SUMMARY = "The Recipe of Copy Config"

FILESEXTRAPATHS:prepend := "${THISDIR}/files:"

SRC_URI += " \
    file://do.sh \
    file://config.conf \
    "

do_install:append() {
	echo "--- VAR ---"
	echo ${THISDIR}
	echo ${D}
	echo ${bindir}
	echo ${D}${bindir}
	echo ${WORK_DIR}
	echo ${datadir}
	echo ${sysconfdir}
	echo "--- EXECUTE ---"
    install -m 0644 ${WORKDIR}/do.sh ${D}${sysconfdir}/
    install -m 0644 ${WORKDIR}/config.conf ${D}${sysconfdir}/rauc-hawkbit-updater/
}
```

Generate updater for bitdove

```bash
# Packing updater at windows MINGW64
# check out bitdove, copy to meta-rauc/recipes-support/rauc-hawkbit-updater/files
@pc1 MINGW64 /c/my/build/git/bitdove-rauc-hawkbit-updater (bitdove)
$ git archive -o "D:\rauc-hawkbit-updater-1.1.tar.gz" 26ee6c48e3d69ff4fe6c4366a1673e11726dee82 --format=tar.gz --prefix=rauc-hawkbit-updater-1.1/

# check out bitdove, copy to meta-rauc/recipes-support/rauc-hawkbit-updater/files
git archive -o "D:\rauc-hawkbit-updater-1.1.tar.gz" HEAD

sha256sum ../meta-rauc/recipes-support/rauc-hawkbit-updater/files/rauc-hawkbit-updater-1.1.tar.gz
b2daaa8a4b035a5393c8318b478c50277a8692b72d86973f459411c8c25d30af  ../meta-rauc/recipes-support/rauc-hawkbit-updater/files/rauc-hawkbit-updater-1.1.tar.gz
```

vi ../meta-rauc/recipes-support/rauc-hawkbit-updater/rauc-hawkbit-updater_1.1.bb

```bash
include rauc-hawkbit-updater.inc

FILESEXTRAPATHS:prepend := "${THISDIR}/files:"

# For local
SRC_URI = "file://rauc-hawkbit-updater-1.1.tar.gz"
# sha256sum rauc-hawkbit-updater-1.1.tar.gz
SRC_URI[sha256sum] = "b2daaa8a4b035a5393c8318b478c50277a8692b72d86973f459411c8c25d30af"
```

vi ../meta-rauc/recipes-support/rauc-hawkbit-updater/rauc-hawkbit-updater.inc

```bash
md5sum LICENSE
10029aac47d30f016272a8568b0743fd

LIC_FILES_CHKSUM = "file://LICENSE;md5=10029aac47d30f016272a8568b0743fd"
```

Integrate ssl for wget

vi ../poky/meta/recipes-core/busybox/busybox.inc

```bash
     busybox_cfg(bb.utils.contains('DISTRO_FEATURES', 'ipv6', True, False, d), 'CONFIG_FEATURE_IFUPDOWN_IPV6', cnf, rem)
     busybox_cfg(bb.utils.contains_any('DISTRO_FEATURES', 'bluetooth wifi', True, False, d), 'CONFIG_RFKILL', cnf, rem)
+    busybox_cfg(True, 'CONFIG_FEATURE_WGET_OPENSSL', cnf, rem)
```

https://github.com/rauc/rauc-hawkbit-updater

```c
// Add 301 redirect
curl_easy_setopt(curl, CURLOPT_WRITEDATA, fetch_buffer);
curl_easy_setopt(curl, CURLOPT_FOLLOWLOCATION, 1L);
```

### Installing The Update Bundle

```bash
bitbake qemu-demo-bundle

rm ~/.ssh/known_hosts

# Keep qemu running
scp -P 2222 tmp/deploy/images/qemux86-64/qemu-demo-bundle-qemux86-64.raucb root@localhost:/data/

cp tmp/deploy/images/qemux86-64/qemu-demo-bundle-qemux86-64.raucb qemu-demo-bundle-qemux86-64-remote-1.raucb

cp tmp/deploy/images/qemux86-64/qemu-demo-bundle-qemux86-64.raucb qemu-demo-bundle-qemux86-64-local-1.raucb

## In device

rauc install /data/qemu-demo-bundle-qemux86-64.raucb

reboot

# check the context updated.
cat /etc/do.sh

rauc-hawkbit-updater -h

cat /etc/rauc-hawkbit-updater/config.conf

rauc-hawkbit-updater -r -d -c /etc/rauc-hawkbit-updater/config.conf

# A/B method, must reboot to check file updated.
reboot
```

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


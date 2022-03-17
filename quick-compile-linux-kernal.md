## Compile linux kernal

Ubuntu 20.04

```bash
wget http://ftp.ntu.edu.tw/linux/kernel/v5.x/linux-5.16.3.tar.xz

tar -xavf linux-*.tar.xz

sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison

make menuconfig

make savedefconfig

make j 4

make modules_install

make install

sudo update-initramfs -c -k 5.14.13

sudo update-grub

sudo reboot -h 0

uname -mrs
```



```bash
wget https://buildroot.org/downloads/buildroot-2021.02.8.tar.gz

tar -xavf buildroot-*

make menuconfig
```



```bash
ERROR: The following required tools (as specified by HOSTTOOLS) appear to be unavailable in PATH, please install them in order to proceed:
  pzstd zstd

sudo apt install -y zstd liblz4-tool

bitbake core-image-minimal
```


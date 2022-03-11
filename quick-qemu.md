# Quick Qemu

## Install

```bash
# Remove apt package if existed
sudo apt-get remove --auto-remove qemu

sudo apt install build-essential
sudo apt install -y ninja-build
sudo apt install -y libglib2.0-dev libpixman-1-dev

wget https://download.qemu.org/qemu-6.2.0.tar.xz
qemu_ver=$(echo qemu-* 2>&1 | grep -oP 'qemu-\K.*(?=\.tar\.xz)')
tar xvJf qemu-$qemu_ver.tar.xz

cd qemu-$qemu_ver
./configure
make -j$(($(nproc) - 1))
sudo make install

cd ..

# Delete all files after installed
rm -R qemu-$qemu_ver
```

## Command

```bash
qemu-system-arm -M vexpress-a9 -m 256 -kernel u-boot -nographic

# hit “Ctrl-a” and then “x” to exit QEMU
```


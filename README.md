# BPI-M2 Zero kernel

Linux kernel based on the mainline 5.17.5 kernel for [Banana Pi M2 Zero](https://wiki.banana-pi.org/Banana_Pi_BPI-M2_ZERO) with WiFi.
Note that the WiFi driver is missing, need to add it via Linux firmware blob(s)

## Build & Install

1. I built this on Ubuntu 22.04 x86\_64 with the [Arm GNU Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads) (arm-none-linux-gnueabihf), so download and extract it and set the bin dir to PATH:

```
export PATH=$PATH:/opt/gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf/bin
```
1.1 NOTE: Seems like that the normal arm-linux-gnueabihf- package from Ubuntu's repos is enough for compiling the kernel.
```
sudo apt install arm-linux-gnueabihf
prepend this to the make command: CROSS_COMPILE=arm-linux-gnueabihf-
```

2. Install the following tools and libs:
```
sudo apt-get install flex bison g++ libgmp3-dev libmpc-dev device-tree-compiler build-essential python3-setuptools python3-dev
```

3. Build from this project root.

```
make INSTALL_MOD_PATH=output ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- m2z_lima_defconfig zImage modules modules_install dtbs -j$(nproc)
```
3.1 Normal arm-linux-gnueabi cross-compiler:
```
make INSTALL_MOD_PATH=output ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- m2z_lima_defconfig zImage modules modules_install dtbs -j$(nproc)
```

4. Copy over the output/* (lib/modules) into rootfs of memory card

```
sudo cp -vfr ./output/lib/* /dev/sda2/lib/
sync
```

5. Install into the /boot dir of the memory card:

```
sudo cp -fv ./arch/arm/boot/zImage /dev/sda1/zImage
sync
sudo cp -fv ./arch/arm/boot/dts/bpi-m2-zero-v4.dtb /dev/sda1/bpi-m2-zero.dtb
sync
```

6. Install/transfer the correct WiFi radio blob to /lib/firmware
```
TODO
```

7. Copy Buildroot/barebones OS rootfs to SDCard for fast booting.
This kernel takes 3s to boot itself, with userspace taking 10s+ to a login shell. (Normal Bookworm rootfs)
```
TODO
```

### Changes from mainline

* (TODO) Enable/add the USB gadget ConfigFS/FunctionFS for g_ether, etc.

* Used the m2z_lima_defconfig from https://github.com/avafinger/linux-5.6.y with some modifications to it. ([This](https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/issues/38#issuecomment-632062680) and some from [here](https://github.com/BPI-SINOVOIP/BPI-M2P-bsp-4.4/blob/b034b7104be40a9fa23a9e8473ef2a1db0d6679c/linux-sunxi/arch/arm/configs/sun8iw7p1smp_bpi-m2z_defconfig#L1821-L1825), although I'm not sure if the latter was necessary for wifi to be working)

* Added [sun8iw7p1smp_bpi-m2z_defconfig](https://github.com/BPI-SINOVOIP/BPI-M2P-bsp-4.4/blob/b034b7104be40a9fa23a9e8473ef2a1db0d6679c/linux-sunxi/arch/arm/configs/sun8iw7p1smp_bpi-m2z_defconfig) and [m2z_defconfig](https://github.com/avafinger/linux-5.6.y/blob/master/arch/arm/configs/m2z_defconfig).

* Added [bpi-m2-zero-v4.dts](https://github.com/avafinger/linux-5.6.y/blob/6dc50035fef1f65b7f5b3b818f69e1e7a3ef0616/arch/arm/boot/dts/bpi-m2-zero-v4.dts), along with the headers it requires and modified the Makefile to accomodate this.

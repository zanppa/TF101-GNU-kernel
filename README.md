# TF101-GNU-kernel

This is a Linux 3.1.10 kernel for Asus Eee Pad Transformer TF101. This on is forked from [mozggg's kernel](https://github.com/mozggg/TF101-GNU-kernel) with some modifications.

## Main changes
 * Reduce I2C bus 2 speed to 10 kHz to stabilize the bus. This hopefully removes the need to periodically re-dock the keyboard.
 * Remove some unneccessary messages from kernel log
 
## Building
### Preparation
1. Cross-compiler Tool
Since this is a pretty old kernel code, we can't compile using latest arm-GCC.
We need to use very specific GCC version (4.7). The easiest is to get pre-built version from here:
```
https://launchpadlibrarian.net/151487636/gcc-arm-none-eabi-4_7-2013q3-20130916-linux.tar.bz2
```
The tools are for 32-bit computers, on x64 it is necessary to also install some libraries (Debian 10):
```
sudo apt-get install libc6-i386 lib32ncurses6
```

### Building
1. Setup environment variables
```
# Tell build system we're compiling for ARM
export ARCH=arm
# Specify tools to do cross-compilation, because we're building for ARM using X86-64
export CROSS_COMPILE=/path/to/downloaded/gcc-arm-none-eabi-4_7-2013q3/bin/arm-none-eabi-
```

2. Use TF101 config as kernel build config
```
make tf101-linux_defconfig
```

3. Build the Kernel
```
make
```

4. Build modules
```
make ARCH=arm modules
```

5. Install modules
Since it most likely is necessary to copy the modules from a SD-card to the target system, we can specify to where install the modules, e.g. to `/tmp/modules`
```
mkdir /tmp/modules
make ARCH=arm INSTALL_MOD_PATH=/tmp/modules modules_install
```

## Create flashable image
1. Install libraries
```
sudo apt-get install libblkid-dev
```

2. Install abootimg
```
git clone https://github.com/ggrandou/abootimg.git
cd abootimg
make
sudo cp abootimg /usr/local/bin/
```

3. Install BlobTools
```
git clone http://github.com/AndroidRoot/BlobTools.git
cd BlobTools
make
sudo cp blobpack blobunpack /usr/local/bin/
```

4. Create the flashable zip
To create a flashable image, easiest is to update an existing one. I use [one from jmrohwer](https://forum.xda-developers.com/showpost.php?p=43177640&postcount=3). Download the kernel .zip file.

unzip the kernel .zip in an empty directory.

Copy the bzImage from /kernel/compile/path/arch/arm/boot/zImage to the same directory.

```
blobunpack blob
abootimg -u blob.LNX -k zImage
rm blob
blobpack blob LNX blob.LNX
zip kernel.zip blob
```

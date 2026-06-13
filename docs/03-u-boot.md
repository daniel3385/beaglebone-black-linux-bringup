# U-Boot Build for BeagleBone Black (AM335x)

## Prerequisites

Install the Device Tree Compiler:

```bash
sudo apt update
sudo apt install device-tree-compiler
```

---

## Download U-Boot

Create a working directory:

```bash
mkdir bootloader
cd bootloader
```

Download U-Boot:

```bash
wget https://ftp.denx.de/pub/u-boot/u-boot-2023.10.tar.bz2
```

Extract the source code:

```bash
tar xvf u-boot-2023.10.tar.bz2
cd u-boot-2023.10
```

---

## Configure Cross-Compilation Environment

Create an environment setup script:

```bash
vim env.sh
```

Example:

```bash
export PATH=$PATH:$HOME/x-tools/arm-cortex_a8-linux-gnueabihf/bin
export CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf-
export ARCH=arm
```

Load the environment:

```bash
source env.sh
```

Verify that the toolchain is available:

```bash
ls $HOME/x-tools/arm-cortex_a8-linux-gnueabihf/bin
```

---

## Configure U-Boot

Use the BeagleBone Black default configuration:

```bash
make am335x_evm_defconfig
```

---

## Build U-Boot

Compile:

```bash
make -j$(nproc)
```

If OpenSSL development files are missing:

```bash
sudo apt install libssl-dev
```

Then run the build again:

```bash
make -j$(nproc)
```

---

## Generated Files

After a successful build, the most important files are:

```text
MLO
u-boot.img
```

Verify they exist:

```bash
ls MLO u-boot.img
```

---

## Copy Files to Windows

From WSL:

```bash
cp MLO /mnt/c/Users/Daniel/Downloads/
cp u-boot.img /mnt/c/Users/Daniel/Downloads/
```

Open the current directory in Windows Explorer:

```bash
explorer.exe .
```

---

## SD Card Boot Files

For BeagleBone Black booting from an SD card, the minimum required bootloader files are:

```text
MLO
u-boot.img
```

These files should be copied to the FAT32 boot partition of the SD card.

---

## Verify the Build

Check the generated binaries:

```bash
file MLO
file u-boot.img
```

Example output:

```text
MLO: ARM executable
u-boot.img: U-Boot legacy uImage
```

---

## Notes

* Board: BeagleBone Black (AM335x)
* U-Boot Version: 2023.10
* Toolchain: arm-cortex_a8-linux-gnueabihf
* Architecture: ARMv7-A
* Floating Point ABI: Hard Float (HF)

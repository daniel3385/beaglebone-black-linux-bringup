# BusyBox RootFS for BeagleBone Black

## Goal

Create a minimal Linux Root Filesystem using BusyBox and boot it on the BeagleBone Black.

At this stage the project already has:

* Custom Crosstool-NG cross compiler
* Custom U-Boot build
* Custom Linux kernel build
* Kernel successfully booting on hardware

The previous milestone was reaching:

```text
Kernel panic - not syncing: VFS: Unable to mount root fs
```

This confirmed that the boot chain is working:

```text
ROM Bootloader
      ↓
MLO
      ↓
U-Boot
      ↓
Linux Kernel
      ↓
Root Filesystem
```

Now we will create the missing userspace.

The final boot flow will become:

```text
U-Boot
    ↓
Linux Kernel
    ↓
Initramfs (CPIO)
    ↓
BusyBox
    ↓
/init
    ↓
Shell
```

---

# Create RootFS Directory Structure

Create the filesystem directory:

```bash
mkdir ~/rootfs
cd ~/rootfs
```

Create the required directories:

```bash
mkdir -p \
bin \
dev \
etc \
proc \
sys \
tmp \
usr/bin
```

Expected structure:

```text
rootfs/
├── bin/
├── dev/
├── etc/
├── proc/
├── sys/
├── tmp/
└── usr/
    └── bin/
```

---

# Build BusyBox

Download BusyBox:

```bash
cd ~

wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2

tar xf busybox-1.36.1.tar.bz2

cd busybox-1.36.1
```

Configure for ARM:

```bash
make ARCH=arm \
CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf- \
defconfig
```

---

# Enable Static Build

Open configuration menu:

```bash
make ARCH=arm \
CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf- \
menuconfig
```

Enable:

```text
Busybox Settings
    └── Build Options
        └── Build BusyBox as a static binary (no shared libs)
```

Disable tc, since it is not compatible with this kernel:

```text
Networking Utilities
    └── tc
```

---

# Compile BusyBox

Build:

```bash
make -j$(nproc) \
ARCH=arm \
CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf-
```

Install into RootFS:

```bash
make CONFIG_PREFIX=~/rootfs install
```

Verify:

```bash
ls -l ~/rootfs/bin
```

Expected:

```text
busybox
sh -> busybox
ls -> busybox
mount -> busybox
```

---

# Create Essential Device Nodes

Linux requires some devices before starting userspace.

Create:

```bash
sudo mknod ~/rootfs/dev/console c 5 1

sudo mknod ~/rootfs/dev/null c 1 3
```

Verify:

```bash
ls -l ~/rootfs/dev
```

Expected:

```text
crw-r--r-- console
crw-r--r-- null
```

---

# Create Init Script

The kernel starts the first userspace process:

```text
/init
```

Create:

```bash
vim ~/rootfs/init
```

Content:

```sh
#!/bin/sh

mount -t proc proc /proc
mount -t sysfs sysfs /sys

echo
echo "================================"
echo " BeagleBone Black RootFS OK"
echo "================================"
echo

exec /bin/sh
```

Make executable:

```bash
chmod +x ~/rootfs/init
```

---

# Generate Initramfs

Enter RootFS directory:

```bash
cd ~/rootfs
```

Create CPIO archive:

```bash
find . | cpio -H newc -ov > ../rootfs.cpio
```

Compress:

```bash
gzip ../rootfs.cpio
```

Generated file:

```text
~/rootfs.cpio.gz
```

---

# Configure Linux Kernel Initramfs

Go to kernel source:

```bash
cd ~/linux-6.6.95
```

Open configuration:

```bash
make menuconfig
```

Enable:

```text
General Setup
    └── Initramfs source file(s)
```

Set:

```text
/home/daniel/rootfs.cpio.gz
```

---

# Rebuild Kernel

Compile kernel and device tree:

```bash
make -j$(nproc) zImage dtbs
```

Generated files:

Kernel:

```text
arch/arm/boot/zImage
```

Device Tree:

```text
arch/arm/boot/dts/ti/omap/am335x-boneblack.dtb
```

---

# Copy Files to SD Card

Copy kernel:

```bash
cp arch/arm/boot/zImage <BOOT_PARTITION>
```

Copy device tree:

```bash
cp arch/arm/boot/dts/ti/omap/am335x-boneblack.dtb <BOOT_PARTITION>
```

---

# U-Boot Environment

`uEnv.txt`:

```text
bootargs=console=ttyO0,115200

loadaddr=0x82000000
fdtaddr=0x88000000

uenvcmd=load mmc 0:1 ${loadaddr} zImage; load mmc 0:1 ${fdtaddr} am335x-boneblack.dtb; bootz ${loadaddr} - ${fdtaddr}
```

The `-` parameter means no external initrd is provided because the RootFS is embedded inside the kernel.

---

# Expected Boot Result

Before adding RootFS:

```text
Kernel panic - not syncing:
VFS: Unable to mount root fs
```

After adding BusyBox initramfs:

```text
================================
 BeagleBone Black RootFS OK
================================

/ #
```

At this point Linux is running a custom userspace created from scratch.

---

# Next Steps

Possible improvements:

* Add network support
* Add SSH server
* Add libraries for dynamic applications
* Add SQLite runtime
* Move RootFS from initramfs to ext4 partition
* Create a complete embedded Linux distribution

# Linux Kernel Bring-Up on BeagleBone Black

## Goal

Boot a custom Linux kernel on the BeagleBone Black using:

* Custom Crosstool-NG toolchain
* Custom U-Boot build
* Custom Linux kernel build
* SD card boot

At this stage no RootFS is provided. The objective is only to verify that the kernel boots successfully and reaches the point where it starts looking for a root filesystem.

Successfully reaching a VFS panic is considered an important milestone because it proves that:

* ROM Bootloader is working
* MLO is working
* U-Boot is working
* Device Tree is valid
* Linux kernel is booting correctly
* Serial console is operational
* Memory initialization is successful

The only missing component is the Root Filesystem.

---

# Download Linux Kernel

Create a working directory:

```bash
mkdir kernel
cd kernel
```

Download a stable Linux kernel release:

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.95.tar.xz
```

Extract sources:

```bash
tar xf linux-6.6.95.tar.xz
cd linux-6.6.95
```

---

# Configure Cross Compiler

Export architecture and toolchain:

```bash
export ARCH=arm
export CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf-
```

Verify toolchain:

```bash
${CROSS_COMPILE}gcc --version
```

---

# Generate Default Configuration

For modern kernels:

```bash
make multi_v7_defconfig
```

---

# Optional: Kernel Customization

Launch menuconfig:

```bash
make menuconfig
```

If an initramfs was previously configured:

```text
General Setup
 └── Initramfs source file(s)
```

Leave this field empty if the goal is only to validate kernel boot.

---

# Build Kernel and Device Tree

Compile:

```bash
make -j$(nproc) zImage dtbs
```

Generated kernel:

```text
arch/arm/boot/zImage
```

Generated Device Tree:

```text
arch/arm/boot/dts/ti/omap/am335x-boneblack.dtb
```

---

# Copy Files to SD Card

Copy kernel image:

```bash
cp arch/arm/boot/zImage <BOOT_PARTITION>
```

Copy device tree:

```bash
cp arch/arm/boot/dts/ti/omap/am335x-boneblack.dtb <BOOT_PARTITION>
```

---

# Configure U-Boot

Create or edit:

```text
uEnv.txt
```

Contents:

```text
bootargs=console=ttyO0,115200

loadaddr=0x82000000
fdtaddr=0x88000000

uenvcmd=load mmc 0:1 ${loadaddr} zImage; load mmc 0:1 ${fdtaddr} am335x-boneblack.dtb; bootz ${loadaddr} - ${fdtaddr}
```

---

# Boot Sequence

```text
ROM Bootloader
    ↓
MLO
    ↓
U-Boot
    ↓
zImage
    ↓
am335x-boneblack.dtb
    ↓
Linux Kernel
    ↓
Root Filesystem Search
```

---

# Expected Result

Because no RootFS is provided yet, the kernel should eventually fail with:

```text
Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
```

Example:

```text
[    4.534237] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
```

---

# Interpretation

This panic is expected.

It confirms that:

* U-Boot successfully loaded the kernel
* U-Boot successfully loaded the Device Tree
* Kernel decompression completed successfully
* Hardware initialization completed
* Linux reached userspace startup phase
* Kernel attempted to locate a root filesystem

At this point the next step is to build a BusyBox RootFS and either:

* Embed it into the kernel as an initramfs
* Boot using an external root filesystem from SD card

```

---

**Milestone achieved:** Custom Linux kernel successfully booted on BeagleBone Black and reached VFS root filesystem initialization stage.
```

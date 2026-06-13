# Crosstool-NG + ARM Cortex-A8 Toolchain (BeagleBone Black)

This document describes the process of building a cross-compilation toolchain using Crosstool-NG for ARM Cortex-A8 (BeagleBone Black) and setting up a basic development environment.

---

## 1. Install required dependencies

```bash
sudo apt update
sudo apt install build-essential git autoconf bison flex texinfo help2man gawk libtool-bin libncurses5-dev unzip
```

---

## 2. Clone Crosstool-NG

```bash
git clone https://github.com/crosstool-ng/crosstool-ng.git
cd crosstool-ng
```

Verify versions available:
```bash
git tag | sort -V | tail
```

Output example:
```bash
crosstool-ng-1.25.0
crosstool-ng-1.25.0-rc1
crosstool-ng-1.25.0-rc2
crosstool-ng-1.26.0
crosstool-ng-1.26.0-rc1
crosstool-ng-1.26.0-rc2
crosstool-ng-1.27.0
crosstool-ng-1.27.0-rc1
crosstool-ng-1.28.0
crosstool-ng-1.28.0-rc1
```

Checkout a stable version:

```bash
git checkout crosstool-ng-1.28.0
```

---

## 3. Build Crosstool-NG

```bash
./bootstrap
./configure --enable-local
make
```
Select CPU closer to BBB as possible
```bash
./ct-ng list-samples | grep cortex_a8
./ct-ng arm-cortex_a8-linux-gnueabi
```
Template is not Hard float, se we need to adjust it using menuconfig:
```bash
./ct-ng menuconfig
```
Change Software FUP to Hardware FPU in the Target options and then write neon in the Specific FPU.
Now we can build:
```bash
./ct-ng build
```

Check binary:

```bash
ls ~/x-tools/arm-cortex_a8-linux-gnueabihf/bin/
```

---

## 4. Setup toolchain PATH

```bash
export PATH=$PATH:$HOME/x-tools/arm-cortex_a8-linux-gnueabihf/bin/
```

Verify compiler:

```bash
arm-cortex_a8-linux-gnueabihf-gcc --version
arm-cortex_a8-linux-gnueabihf-gcc -v
```

Check sysroot:

```bash
arm-cortex_a8-linux-gnueabihf-gcc -print-sysroot
```

---

## 5. Inspect sysroot

```bash
cd $HOME/x-tools/arm-cortex_a8-linux-gnueabihf/arm-cortex_a8-linux-gnueabihf/sysroot

ls
ls usr/
ls usr/include
ls usr/lib
```

---

## 6. Cross-compiling a simple library (SQLite example)

Download source:

```bash
wget https://sqlite.org/2020/sqlite-autoconf-3330000.tar.gz
tar xf sqlite-autoconf-3330000.tar.gz
cd sqlite-autoconf-3330000
```

Configure for cross compilation:

```bash
CC=arm-cortex_a8-linux-gnueabihf-gcc ./configure --host=arm-cortex_a8-linux-gnueabihf --prefix=/usr
```

Build:

```bash
make
```

Install into sysroot:

```bash
make DESTDIR=$(arm-cortex_a8-linux-gnueabihf-gcc -print-sysroot) install
```

---

## 7. Verify installation in sysroot

```bash
cd $(arm-cortex_a8-linux-gnueabihf-gcc -print-sysroot)

ls lib
ls usr/lib
ls usr/include
```

### 8. Create simples app using sqlite

```C
#include <stdio.h>
#include <sqlite3.h>

int main(void)
{
    printf("SQLite version: %s\n", sqlite3_libversion());
    return 0;
}
```
Compile and check the binary
```bash
arm-cortex_a8-linux-gnueabihf-gcc test.c -lsqlite3 -o test
file test
```
Shoul be something like:
```bash
test: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 6.16.0, with debug_info, not stripped
```


---

## Notes

- Always ensure `CROSS_COMPILE` matches your toolchain prefix:
  ```bash
  export CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf-
  ```
- Always specify:
  ```bash
  make ARCH=arm CROSS_COMPILE=...
  ```
- Crosstool-NG simplifies embedded Linux development significantly.

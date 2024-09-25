---
title: "How to Debug Linux Kernel (with `qemu` & `busybox`)"
date: 2024-09-25T17:00:55-05:00
draft: false
toc: false
images:
tags:
  - kernel
---

## Outline

- clone `linux` source code
- clone `busybox`
- check compile configs
- generate `initramfs` by `busybox`
- install `qemu`
- boot kernel by `qemu` attached with user space from `busybox`
- other configurations

## Prepare Linux src

> before get started, make sure you have `flex` & `bison` installed to config and compile linux kernel

- clone [linux src](https://github.com/torvalds/linux.git)

```bash
git clone https://github.com/torvalds/linux.git
cd linux
```

- config it for minimal:

```bash
make allnoconfig
```

- and enable following config by:

```bash
make menuconfig
```

> 64-bit kernel ---> yes
> General setup ---> Initial RAM filesystem and RAM disk (initramfs/initrd) support ---> yes
> General setup ---> Configure standard kernel features ---> Enable support for printk ---> yes
> Executable file formats / Emulations ---> Kernel support for ELF binaries ---> yes
> Executable file formats / Emulations ---> Kernel support for scripts starting with #! ---> yes
> Device Drivers ---> Generic Driver Options ---> Maintain a devtmpfs filesystem to mount at /dev ---> yes
> Device Drivers ---> Generic Driver Options ---> Automount devtmpfs at /dev, after the kernel mounted the rootfs ---> yes
> Device Drivers ---> Character devices ---> Enable TTY ---> yes
> Device Drivers ---> Character devices ---> Serial drivers ---> 8250/16550 and compatible serial support ---> yes
> Device Drivers ---> Character devices ---> Serial drivers ---> Console on 8250/16550 and compatible serial port ---> yes
> File systems ---> Pseudo filesystems ---> /proc file system support ---> yes
> File systems ---> Pseudo filesystems ---> sysfs file system support ---> yes
> Processor type and features ---> Linux guest support ---> Support for running PVH guests ---> yes

you may check `linux/.config` to see whether its configured successfully

- then compile it:

```bash
make -j
```

## Prepare BusyBox

- clone [BusyBox](https://github.com/mirror/busybox.git)

```bash
git clone https://github.com/mirror/busybox.git
```

- config it:

```bash
make defconfig
```

- we're willing to compile it statically.
  so we need to enable the following features by:

```bash
make menuconfig
```

> Settings ---> Build Options ---> Build static binary

- then generate the rootfs by:

```bash
make install
```

- modify rootfs:
  > create a file named `init` at the root of **\_install** under busybox with following content:

```bash
#!/bin/sh 

mkdir /proc
mkdir /sys

mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev 

echo "Booting BusyBox..."
exec /bin/sh
```

- create initramfs by:

```bash
cd _install
find . | cpio -H newc -o | gzip > ../ramdisk.img
```

## Install qemu

```bash
sudo apt install qemu-kvm
```

## Boot the kernel and user space

```bash
qemu-system-x86_64 -nographic --append "console=ttyS0" -kernel ./vmlinux -initrd ../busybox/ramdisk.img
```

## Other Configs

- before doing other configs, you may need to use default config
- copy example `inittab` file

```bash
mkdir busybox/_install/etc
cp busybox/example/inittab busybox/_install/etc/inittab
```

**remember to regenerate ramdisk after any modification done to files in this folder**

### tty not available keep scrolling

- comment these lines in `etc/inittab`

```inittab
tty2::askfirst:-/bin/sh
tty3::askfirst:-/bin/sh
tty4::askfirst:-/bin/sh

# /sbin/getty invocations for selected ttys
tty4::respawn:/sbin/getty 38400 tty5
tty5::respawn:/sbin/getty 38400 tty6
```

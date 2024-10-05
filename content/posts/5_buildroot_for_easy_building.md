---
title: "Buildroot for Easy Building"
date: 2024-10-05T17:17:31-05:00
draft: false
toc: false
images:
tags:
  - kernel
---

## Outline

- build `buildroot`
- customization by `overlay`

## Introduction

as in [Debug Linux Kernel](./2_debug_linux_kernel.md), we used `busybox` for building root fs, which need many customization to work properly.
Here, we're going to use `buildroot` to generate the rootfs for us, instead of compile `busybox` by ourselves

## Build `Buildroot`

```bash
git clone https://github.com/buildroot/buildroot.git
make menuconfig
```

> edit:
> target options ---> target architecture
> filesystem images ---> cpio the root filesystem
> enable other packages, I need to use `numactl`, ...
> we need to use a customized linux kernel, we only need the rootfs from buildroot, so disable kernel and bootloader in config

then in the `buildroot/output/images` folder, the ramdisk is generated as: `rootfs.cpio`

it can be booted with:

```bash
 qemu-system-x86_64 -nographic -kernel ../../../linux/vmlinux -initrd rootfs.cpio -append "console=ttyS0"
```

> login as `root` with `no password`

## Customization by Overlay

In my case, I want to have a copy of my own kernel module, which is built out of tree, in the rootfs.

> in `menuconfig`, enable:
> system configuration ---> root filesystem overlay directories
> -- in this case, we use `rootfs_overlay` as directory

then, anyfile in this folder will be copied to the image in relative folder after `make`

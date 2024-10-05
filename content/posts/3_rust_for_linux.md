---
title: "Rust for Linux"
date: 2024-09-27T10:53:59-05:00
draft: false
toc: false
images:
tags:
  - kernel
  - rust
---

**please check previous post [Debug Linux Kernel](./2_debug_linux_kernel.md), make sure you know how to debug linux kernel before starting trying to use rust for linux kernel**

other details are documented in `linux/Documentation/rust/quick-start.rst`

## Outline

- compile `linux` with `rust`
- sample code
- rust-analyzer
- hints
- debug

## Compile Linux with Rust Enabled

**note: rust is added to mainline after `6.1`, so make sure you are using linux above this version**

> make sure you have `llvm`, `clang`, `lld`, `rust` installed before this section

as rust use `llvm` tool chain, we need to configure and compile linux with `llvm`:

- configure linux with `Rust` and `LLVM`, for minimal config:

```bash
make LLVM=1 allnoconfig
```

- enable some essential features by:

```bash
make LLVM=1 menuconfig
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

- we need to fix rustc version to the version required by kernel:

```bash
rustup override set $(scripts/min-tool-version.sh rustc)
```

- check if `rust` is available:

```bash
make LLVM=1 rustavailable
```

until it says: Rust is available!

and to fix all the error it yield to you, you may need to do:

```bash
rustup component add rust-src
cargo install --locked bindgen-cli
```

- then, enable all the `rust-related` feature by:

```bash
make LLVM=1 menuconfig
```

> General setup ---> Rust support ---> yes
> Kernel hacking ---> Rust hacking ---> Debug assertions ---> yes

- compile kernel:

```bash
make LLVM=1 -j
```

## Sample code

- enable `rust` sample code in kernel config by:

```bash
make LLVM=1 menuconfig
```

> Kernel hacking ---> Sample kernel code ---> Rust samples

**and the sample code is located at `linux/samples/rust/`**

you can refer to it for how to develop kernel module in `rust`

## Rust Analyzer

compile kernel with `rust-analyzer` support

```bash
make LLVM=1 -j rust-analyzer
```

which will give me: `rust-project.json`

## Hints

- **always rustfmt before commit**

```bash
make LLVM=1 -j rustfmt
make LLVM=1 -j rustfmtcheck
```

- **always generate document**

```bash
make LLVM=1 -j rustdoc
```

- **use clippy**

```bash
make LLVM=1 -j CLIPPY=1
```

- **test code unrelated with kernel**

```bash
make LLVM=1 -j rusttest
```

## Test

- enable `KUNIT`

```bash
make LLVM=1 menuconfig
```

> Kernel hacking ---> Kernel Testing and Coverage ---> Kunit
> Kernel hacking ---> Rust hacking ---> doctests for `kernel` crate

- qemu will run the tests when booting

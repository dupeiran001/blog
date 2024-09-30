---
title: "Rust for Kernel Module"
date: 2024-09-27T11:24:03-05:00
draft: false
toc: false
images:
tags:
  - kernel
  - rust
---

## Outline

- about kernel module
- add a new module in `rust`

## About Kernel Module

### Lifetime of kernel module

- init when being `loaded`
- register to the certain `subsystem`

- when receive some action from an `actor`
- subsystem perform a `callback` to the module
- then returns

......

- when `unload`
- unregister to `subsystem`

### What we're going to build

some `/dev/scull0`, `/dev/scull1`, ...
that are **readable** and **writeable**
**TODO**

### Development workflow

- do some change to kernel src
- boot with `qemu` with `busybox`, resources can be found [Debug Linux Kernel](./debug_linux_kernel.md) & [Rust for Linux](./rust_for_linux.md)
- and debug it through remote `gdb`

## Add a New Module

### modify Kconfig and Makefile

- add the `Kconfig` at `linux/samples/rust/Kconfig` for our new module:

```Kconfig
config SAMPLE_RUST_SCULL
	tristate "Scull module"
	help
	  This option builds the Rust scull module sample.

	  To compile this as a module, choose M here:
	  the module will be called rust_scull.

	  If unsure, say N.
```

- add the compile command for our new module at `linux/samples/rust/Makefile`:

```Makefile
obj-$(CONFIG_SAMPLE_RUST_SCULL)			+= rust_scull.o
```

- create the file

```bash
touch linux/samples/rust/rust_scull.rs
```

- enable this in `menuconfig`
- then it should compiles
- remember to use `make LLVM=1 rust-analyzer` to generate index for lsp

### Edit our module

in `linux/samples/rust/rust_scull.rs`

```rust
//! Scull module in Rust

use kernel::prelude::*;

module! {
    type: Scull,
    name: "scull",
    license: "GPL",
}

struct Scull;

impl kernel::Module for Scull {
    fn init(_module: &'static ThisModule) -> kernel::error::Result<Self> {
	    pr_info!("Hello World\n");
        Ok(Scull)
    }
}
```

now it can be compiled, you can see the hello world from our module by `dmesg`

---
layout: post
title: An experience on booting aarch64 kernel on qemu
categories: Linux
tags: [qemu, kvm, kernel, aarch64, arm]
---

## In the front

This blog talks about an experience on booting aarch64 kernel on qemu/kvm.  The initial purpose for doing this is to setting up an environment to debugging the kernel with an external debugger.  However, during setting up qemu booting, I ran into some issues that worth logging here.

The basic requirements for booting a simple kernel on qemu contains three parts.  1, emulation/virtualization tool, i.e. qemu.  2, linux kernel cross compiled for aarch64 (armV8).  3, root filesystem for utilities and programs after booting the kernel. (providing a simple shell and fs).

The problem I met turns out to be a version issue for tools used by those three parts.  At first, I used the qemu and cross-compiler from the OS distribution (Ubuntu 16.04).  However, after the kernel is built and booting on qemu, it has no response (whether there is graphic enabled or not, no booting message at all).  I thought this might be an issue for the cross compiler and I tried to compile a cross-compiler with the latest version (which is a pain).  However, the problem turns to be the qemu version too old.  I'll show the references and run scripts for each step in the following sections.

## 1, Cross-compiler for aarch64

Building the gcc is non-trival, especially for the cross-compiler.  I'll not show detailed steps, here is an [article](https://preshing.com/20141119/how-to-build-a-gcc-cross-compiler/) that basically covered everything.

Noted except for gcc compiler, there's another important package which is binutil which contains the linker and loader for aarch64 which also needs to be cross compiled separately.  If you need the full toolchains to cross-compile (like a user program), you also need the proper c/c++ library (glibc) to be cross compiled.

## 2, Building the kernel
Building the kernel is trival and personally I've done this many times for different targets.  For aarch64 there are few notes to mention.

1. the easiset way to config is using the defconfig as follows.

```shell
ARCH=arm64 make defconfig
```

2. to add an external initramfs, complete the configuration line: (buildroot will be explained in 
section 3)

```shell
CONFIG_INITRAMFS_SOURCE="/your/path/buildroot-2019.08.3/output/images/rootfs.cpio"
```

3. build command (depends on which cross-compiler you are using)

```shell
ARCH=arm64 CROSS_COMPILE=/opt/cross/bin/aarch64-linux- make -j16
```

## 3, Buildroot

Buildroot is a useful tool for building small root file system image.  It supports multiple image format and utilities.  Below are some useful configurations.
```shell
* Target Options -> Target Architecture(AArch64)
* Toolchain -> Toolchain type (External toolchain)
* Toolchain -> Toolchain (Linaro AArch64 14.02)
* System configuration -> Run a getty (login prompt) after boot (BR2_TARGET_GENERIC_GETTY)
* System configuration -> getty options -> TTY Port (ttyAMA0) (BR2_TARGET_GENERIC_GETTY_PORT)
* Target Packages -> Show packages that are also provided by busybox (BR2_PACKAGE_BUSYBOX_SHOW_OTHERS)
* Filesystem images -> cpio the root filesystem (for use as an initial RAM filesystem) (BR2_TARGET_ROOTFS_CPIO)
```
The output image is in
```shell
output/images/rootfs.cpio
```
which should be put into the kernel config to linked to the kernel binary.

## 4, QEMU
It's important to use an up-to-date QEMU in able to successfully booting up the kernel.  In my case I used version 4.2.0.  In order to make shared folder between host and guest works, you need to enable virtfs feature (described below).  The configuration flags (for aarch64 softmmu only) are:
```shell
mkdir build; cd build
../configure --enable-virtfs --target-list=aarch64-softmmu
```

The commands for booting an aarch64 kernel is as follows:
```
./aarch64-softmmu/qemu-system-aarch64 \
-M virt \
-kernel ../../linux-4.9.1/arch/arm64/boot/Image \
-nographic -cpu cortex-a53 \
-fsdev local,id=test_dev,path=/home/grads/c/celery1124/shared,security_model=none \
-device virtio-9p-pci,fsdev=test_dev,mount_tag=test_mount
```
## 5, VIRTFS (9P_FS)
9P_FS can enable host and guest shared folder which is useful for transfering data between host and guest.  For example, you can cross-compile code and run in guest easily.
This [article](https://wiki.qemu.org/Documentation/9psetup) documents how to setup and mount shared folder in guest in qemu.

### Other useful references
[https://github.com/google/syzkaller/blob/master/docs/linux/setup_linux-host_qemu-vm_arm64-kernel.md](https://github.com/google/syzkaller/blob/master/docs/linux/setup_linux-host_qemu-vm_arm64-kernel.md)

[https://www.bennee.com/~alex/blog/2014/05/09/running-linux-in-qemus-aarch64-system-emulation-mode/](https://www.bennee.com/~alex/blog/2014/05/09/running-linux-in-qemus-aarch64-system-emulation-mode/)

[https://blukat29.github.io/2017/12/cross-compile-arm-kernel-module/](https://blukat29.github.io/2017/12/cross-compile-arm-kernel-module/)

[https://zhuanlan.zhihu.com/p/53325393](https://zhuanlan.zhihu.com/p/53325393)
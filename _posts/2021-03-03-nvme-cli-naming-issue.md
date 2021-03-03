---
layout: post
title: nvme-cli naming issue
categories: Storage
tags: [linux, storage, nvme, kvssd]
---

## Problem
Recently I was working on a couple of projects related the new key-value interfaced storage device (Samsung KV-SSD). The KV storage devices are adopted in the nvme standard and can be used simply as a nvme device. However, currently the nvme block devices and nvme kv devices use separated nvme drivers (nvme-core, nvme). 

The problem I got with the nmve-cli naming issue is that whenever I reload the nvme related drivers, the device names showed under nvme-cli (/dev/nvme0n1 for example) is different from the actual device under /dev/nvmeX. The normal operation with the nvme device such as make partition, etc. can be done under /dev/nvmeXnY from nvme-cli without any issue.  However, if I need to download firmware on a device from multiple devices attached on the system. It may re-firmware the wrong drive!!!

## Resolution
At first, I simply physically access the machine I use and pull out the drives except for the one I am going to update firmware on.  Then the system only show one drive from /dev/nvmeX.  This is pain since I have to physically access the machine every time I need to do a firmware update.  Luckily, I found an easier route from a post on the nvme-cli github repo.  From the following command you can find the associated nvme namespace for that device.

```bash
ls /sys/class/nvme/nvmeX/
```

### Other useful references
[https://github.com/linux-nvme/nvme-cli/issues/510](https://github.com/linux-nvme/nvme-cli/issues/510)

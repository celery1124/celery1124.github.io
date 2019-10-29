---
layout: post
title: A Debugging Experience on Cortex-A57
categories: Debug
tags: [cortex-a57, fpga, cache coherence]
---

I want to keep record of a debugging experience for cache coherence problem on the ARM cortex-A57 platform for future reference.  We had a project for hardware-based memory management for hybrid memory systems (fast expensive DRAM + slow but cheap NVM).  The hardware platform we are using is an EMC customized board with an 8 core ARM A57 connected with FPGA controlled DIMMs through PCIe bus as shown in Figure 1.  The memory management logic is implemented in FPGA.  The ARM chip serves as host to run applications.  We assume in the future, the processor will have extra logic along with the memory controller for DRAM and NVM to implement some page/sub-page movement policy between DRAM and NVM.

|![Platform diagram](/post_imgs/emc-board-block-diagram.png){:class="img-responsive"}|
|:--:|
| **Figure 1 Platform diagram** |

I was resposible for the host side, i.e. mapping the application memory allocate to the I/O memory (PCIe bar memory).  We developed a customized driver to make the memory-mapped IO cacheable and modified **jemalloc** to hook up with the application code.  The other student works on the RTL to implement the policy of page movement between two tiers of memory.  As the project develops, we might a lot of issues when running the real applications (SPEC 2006).  I always think this is a hardware implementation bug since there are a lot of changing going on during that time.

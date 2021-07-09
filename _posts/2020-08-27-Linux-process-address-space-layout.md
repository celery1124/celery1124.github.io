---
layout: post
title: Linux process address space layout
categories: [Operating system]
tags: [linux, memory, address space, operating system]
---


In this post, I will demonstrate the linux process address space layout, which is also a popular interview questions that can be digged deeply.  Here we focus on modern x86_64 rather than 32 bits legacy x86 mode.

## Process memory layout

For each process in linux, it has user-space virtual memory and kernel-space virtual memory.  For all the process, the kernel-space virtual memory is shared (identical).  This is done through sharing half of the page directory entry in the page table directory.

### kernel space layout

Linux [document](https://www.kernel.org/doc/Documentation/x86/x86_64/mm.txt) gives details description of how kernel-space virtual memory is laied out as below.  An **important region** is the direct mapping region which mapps the entire physical address to virtual address continously (in 32bit mode there are only ~800MB are mapped).  This can be done efficiently through large page mapping, say 1GB large page.  Doing so, the physical address is easily accessed through MMU (by adding an offset to the physical address we get virtual address).  This is important to modify some kernel structure like page table.

``` plain
========================================================================================================================
    Start addr    |   Offset   |     End addr     |  Size   | VM area description
========================================================================================================================
                  |            |                  |         |
 0000000000000000 |    0       | 00007fffffffffff |  128 TB | user-space virtual memory, different per mm
__________________|____________|__________________|_________|___________________________________________________________
                  |            |                  |         |
 0000800000000000 | +128    TB | ffff7fffffffffff | ~16M TB | ... huge, almost 64 bits wide hole of non-canonical
                  |            |                  |         |     virtual memory addresses up to the -128 TB
                  |            |                  |         |     starting offset of kernel mappings.
__________________|____________|__________________|_________|___________________________________________________________
                                                            |
                                                            | Kernel-space virtual memory, shared between all processes:
____________________________________________________________|___________________________________________________________
                  |            |                  |         |
 ffff800000000000 | -128    TB | ffff87ffffffffff |    8 TB | ... guard hole, also reserved for hypervisor
 ffff880000000000 | -120    TB | ffff887fffffffff |  0.5 TB | LDT remap for PTI
 ffff888000000000 | -119.5  TB | ffffc87fffffffff |   64 TB | direct mapping of all physical memory (page_offset_base)
 ffffc88000000000 |  -55.5  TB | ffffc8ffffffffff |  0.5 TB | ... unused hole
 ffffc90000000000 |  -55    TB | ffffe8ffffffffff |   32 TB | vmalloc/ioremap space (vmalloc_base)
 ffffe90000000000 |  -23    TB | ffffe9ffffffffff |    1 TB | ... unused hole
 ffffea0000000000 |  -22    TB | ffffeaffffffffff |    1 TB | virtual memory map (vmemmap_base)
 ffffeb0000000000 |  -21    TB | ffffebffffffffff |    1 TB | ... unused hole
 ffffec0000000000 |  -20    TB | fffffbffffffffff |   16 TB | KASAN shadow memory
__________________|____________|__________________|_________|
                  |            |                  |         |
 fffffc0000000000 |   -4    TB | fffffdffffffffff |    2 TB | ... unused hole
                  |            |                  |         | vaddr_end for KASLR
 fffffe0000000000 |   -2    TB | fffffe7fffffffff |  0.5 TB | cpu_entry_area mapping
 fffffe8000000000 |   -1.5  TB | fffffeffffffffff |  0.5 TB | ... unused hole
 ffffff0000000000 |   -1    TB | ffffff7fffffffff |  0.5 TB | %esp fixup stacks
 ffffff8000000000 | -512    GB | ffffffeeffffffff |  444 GB | ... unused hole
 ffffffef00000000 |  -68    GB | fffffffeffffffff |   64 GB | EFI region mapping space
 ffffffff00000000 |   -4    GB | ffffffff7fffffff |    2 GB | ... unused hole
 ffffffff80000000 |   -2    GB | ffffffff9fffffff |  512 MB | kernel text mapping, mapped to physical address 0
 ffffffff80000000 |-2048    MB |                  |         |
 ffffffffa0000000 |-1536    MB | fffffffffeffffff | 1520 MB | module mapping space
 ffffffffff000000 |  -16    MB |                  |         |
    FIXADDR_START | ~-11    MB | ffffffffff5fffff | ~0.5 MB | kernel-internal fixmap range, variable size and offset
 ffffffffff600000 |  -10    MB | ffffffffff600fff |    4 kB | legacy vsyscall ABI
 ffffffffffe00000 |   -2    MB | ffffffffffffffff |    2 MB | ... unused hole
__________________|____________|__________________|_________|___________________________________________________________
```

### user space layout

User space memory layout is as follows. The **text**, **data**, **bss** is loaded from ELF binary during [exec](https://man7.org/linux/man-pages/man3/exec.3.html) system call to initlize the process.  The heap area is growing upwards and stack area is growing downwards.  

``` plain
User Stack
    |
    v
Memory Mapped Region for Shared Libraries or Anything Else
    ^
    |
Heap
Uninitialised Data (.bss)
Initialised Data (.data)
Program Text (.text)
0000000000000000
```

For stack, things like call stack and local variables are stored there.  It is managed through instruction level by manipulating the frame pointer register (ebp), stack pointer registe (esp) and push/pop/call/ret instructions.  If the address is beyond the mapped address in the page table, a page fault should ocurr and a frame will be allocated and mapped to the page table for the stack area.  For heap management, it's usually managed through system call [brk](https://man7.org/linux/man-pages/man2/brk.2.html) and it's familiy.  A lot of cases when allocating memory through **malloc**, memory is allocated through [mmap](https://man7.org/linux/man-pages/man2/mmap.2.html) which will be mapped to the memory mapped region between stack and heap region.

## How kernel manage memory region

Kernel manages the memory regions through the **vm_area_struct** in the process description struct.  It can be structured using linked list or tree for easy access.  The vm_area_struct is checked during protection fault and modified when adding/deleting memory regions (like a memory mapped area).

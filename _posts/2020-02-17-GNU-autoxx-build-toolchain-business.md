---
layout: post
title: GNU autoxx build toochain business
categories: GNU
tags: [GNU, autotools, autoconf, automake]
---

This blog briefly talks about the GNU build toochain, (i.e. autoconf, automake) and how to use it.  As a long time Linux user.  Probably we all know the below build process for something that cannot retrieve binary directly from package manager but needs to be compiled from source.

``` bash
./configure --prefix=$DIR # DIR is the place you want to install
make -j4
make install
```

However, it's always a mystery where the **configure** script comes from (we know that configure will generate Makefile).  Recently, I got a change to touch the GNU build system by working on a project that profile the heap memory access.  Valgrind already provides a tool called **dhat** that can annotate (providing the stack frame in code level) the malloc/new region and profile the read/write bytes on each region.  However, in this project (we are working on a hardware memory manager for heterogeneous memory system), we want to profile the memory request on off-chip memory (hybrid DRAM NVM).  To better demonstrate the background, we've already built a system on an ARM-FPGA card that boot Linux on ARM and mapped the application heap on two channels of FPGA DRAM (through memory mapped IO).  In order to profile the memory access to the off-chip memory, I merged the other tool Valgrind provide, i.e. cachegrind with dhat and made [**cachedhat**](https://github.com/celery1124/valgrind-cachedhat) to profile the heap memory request to off-chip memory.

During working on this project, I need to touch on the Valgrind build system and get to know a bit more on [autotools suite]((http://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html#Autotools-Introduction)) (containing autoconf and automake) toolchain.  In a nutshell (actually I didn't get much details but at least go through the whole process).  There are two manually edit scripts for generating the configure and further Makefile.  First, configure.ac is the source script for auto generate configure.  For example, in Valgrind, I need to specify the new generated Makefiles in the added cachedhat tool.  Second, Makefile.am, this is the input for automake to generate the Makefile.in script for configure script to further generate Makefile.  Similar like the CMakeLists.txt, in Makefile.am you need to specify the source files and compile flags in order to generate the final Makefile.

After all these words, here I'll show in general how GNU build works.

On the maintainer's machine (like here I add a new tool to Valgrind).

``` bash
aclocal # Set up an m4 environment
autoconf # Generate configure from configure.ac
automake --add-missing # Generate Makefile.in from Makefile.am
./configure --prefix=$DIR # Generate Makefile from Makefile.in
make ; make install
```

After getting the source code, the user can do the same configure and make business to build and install the project.

## Reference
[https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install](https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install)

[http://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html#Autotools-Introduction](http://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html#Autotools-Introduction)
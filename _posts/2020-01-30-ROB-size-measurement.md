---
layout: post
title: ROB size measurement through real machine
categories: Architecture
tags: [ROB, interview, microarchitecture]
---

A interesting interview question for architecture positions is how to measure the instruction window size.  To answer this question, you need to have a full understanding of the modern OOO microarchitecture. Instruction window refers to the maximum number of instructions which can execute out-of-order in a processor.  It is bounded by the reorder buffer (ROB) size.  The reorder buffer is used for in-order instruction commit to implement precise exception.  In the pipeline, during assigning entry to the reservation station, a entry in reorder buffer will be allocated and the destination register will be renamed with the reservation station entry tag (basically a reservation station entry ID).  Once the instruction is executed, it will broadcast to ROB and reservation station to notify the finish of the instruction and data dependancy.  The ROB will be constatantly checked and deallocated in-order to commit the instructions **in-order**.  

So, back to the question, how should we measure the instruction window size or, ROB size?  The idea is to use latency measurement of a load instructions (make sure it's not hit in cache).  If multiple loads fell in the reorder buffer, the latency will be hidden due to load parallelsim.  The trick is to stuff some instructions (nops for example) between load instructions and by tweaking the number of stuffed instructions between two load instructions and measure the average load latency (again, make sure the load is missed from cache).  You should oberseve a sharp increase of the latency (only 1 load instruction filled in ROB).  The changing point can tell you the ROB size (how many stuffed instructions).

Credit to this [blog](https://blog.stuffedcow.net/2013/05/measuring-rob-capacity/).  It provides many insteresting architecture related articles.  
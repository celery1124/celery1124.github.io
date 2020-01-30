---
layout: post
title: Cache Prefetching Policy Summary
categories: Architecture
tags: [cache, prefetch]
---

# Overview

In this blog, I will summarize some non-trivial famous cache prefetch techniques (hardware) in computer architecture.  Figure 1 demonstrates the memory hierarchy on modern CPUs.  From the absolute access latency for each level, you can see the potential of hardware data prefetching (since usually cache are transparent to software). 
|![Modern memory hierarchy](/post_imgs/modern_memory_hierarchy.png){:class="img-responsive"}|
|:--:|
| **Figure 1 Modern memory hierarchy[[1]][A Primer on Hardware Prefetching]** |

# Policies

In this section, we categorize based on the policy names.

## Instruction cache prefetch

Fit cache level: L1 ICache

Since the focus in this blog is data prefetching, here I only use single section to talk about the instruction cache prefetch.  Since instruction fetch is most correlated with PC.  Thus, it has relatively obvious approach to guide prefetch.  

[Next line][next line] prefetcher is the earliest prefetch techniques that is first use by IBM processor.  The idea of next line prefetch is the observation of sequential access of the code (within a routine).  So, it can blindly prefetch the next block during instruction cache access.  Noted in the actual implementation of prefetcher, it implemented with a small associate buffer (prefetch buffer or stream buffer, basically a single cache set) for the prefetched block.  The fetch unit will parallel access the prefetch buffer and L1 instruction cache.  This design is common for instruction cache prefetcher.

Another improvement design for instruction cache prefetcher is using branch predictor to guide the prefetch decision, which is obvious since branch prediction has a "future" view of the control flow (non sequential code address in the BTB).  [Fetch-directed instruction prefetching][FDIP] introduce two buffers, one prefetch target buffer, one prefetch buffer (also a prefetch queue).  The prefetch target buffer bridge the gap between branch predictor and instruction cache fetch.  The prefetch buffer is parallel accessed with the I cache.  Also, the paper mentioned to flush the two buffers when mis-predicts.

The above prefetcher cannot achieve above 95% accuracy.  Next line depends on the sequentiality of the code (usually only 50% of the code has sequentiality).  Branch prediction guide prefetch limited by the PC (PC can only direct to single branch target address).  Thus, simply using PC is not the best idea rather than directly using the instruction cache block address (this is going through the same direction as data prefetching).  [Temporal instruction fetch streaming][TIFS] uses the pure address rather than PC to guide prefetch.  It index the L1 Instruction cache miss address to instruction miss log and can prefetch multiple blocks based on history misses.

In summary, the instruction cache prefetcher is strongly correlated with the code control flow (functional call, return address, etc.).  Thus, it has less difficulty to perform very high accuracy for the prefetch (more than 99%).  However, due to the much more variety of data access pattern which is correlated to the application itself.  Data prefetch is a more challenging and interesting topic which will be covered in the next sections.  

## Stride and Streaming prefetch

Stride stands for load address stream with fixed offset (next line is a special case).  Streaming means prefetch more than 1 blocks in a prefetch **stream**.  Together it forms a class of prefetcher called Stride and Streaming prefetcher.  Baer and Chen proposed a [prefetcher][stride and streaming] in this class.  It mainly contains a table with PC indexed, including the information of the last load address referenced and stride length (offset) and a single predict bit.  When the stride is well trained and predicted, the prefetcher can initiate prefetch on the next stride/s when the load misses and the PC is matched.

Another important aspect of stride and streaming prefetch is the streaming length or prefetch degree/depth.  There are ideas to [predict the stream length][Memory Prefetching Using Adaptive Stream Detection] to make the prefetch more accurate

Stride and streaming method is highly depends on data access pattern.  For example, most of the paper use the matrix multiply as an example since it contains multiple loop with strong stride access pattern.  For other more complex access pattern like pointer chasing, this will definitely not work.

## Best-offset prefetch

## SMS

## SPP


[A Primer on Hardware Prefetching]:https://www.amazon.com/Hardware-Prefetching-Synthesis-Architecture-Paperback/dp/B011YU7GFQ

[next line]:https://ieeexplore.ieee.org/document/1646791

[FDIP]:https://dl.acm.org/doi/10.5555/320080.320085

[TIFS]:https://dl.acm.org/doi/10.1109/MICRO.2008.4771774

[stride and streaming]:https://dl.acm.org/doi/10.1145/125826.125932

[Memory Prefetching Using Adaptive Stream Detection]:https://ieeexplore.ieee.org/document/4041863
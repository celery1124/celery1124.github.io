---
layout: post
title: MESI protocol for cache coherence and more
categories: Architecture
tags: [coherence, cache coherence, MESI]
---

Recently I was interviewed with a design question regarding coherence. The problem statement is like this.  We have a server that hold the ground truth of the data (like a database or key-value store).  We also have many clients that may read the data from the sever and make local updates to the data.  What kind of design guidence should we consider for such system.  This is acutally a similiar problem as cache coherence for multi-core system.  In this blog I will reveiw the classic MESI protocol for cache coherence.

## Overview

|![MESI state diagram](/post_imgs/MESI.jpg){:class="img-responsive"}|
|:--:|
| **Figure 1 MESI state diagram[[1]]** |

The general approach to implement cache coherence is the SNOOPY based methods. The idea is to have a common bus connecting the private caches and the shared next level cache or main memory.  The basic protocol is a valid/invalid protocol that only implement two states.  Whenever a core miss on a block, it will invalidate all the other cores that hold the block and mark it valid.  Building upon the valid/invalid protocol is the MSI protocal which differ clean and dirty state of a cache.  Transfer to **M**odified state will invalidate the block from other cores.  **S**hared state will also snoop on the bus but may not invalidate block if there is no M block on the other cores.  However, there is a problem for the MSI protocol when a core write to the shared state block (from shared to modified).  It always needs to broadcast message to other cores even it's the only core hold the block.  To avoid this problem, MESI protocol is proposed as shown in Figure 1.  The following section will elabrate the state transfer of the MESI protocol.

## MESI State Transfer

Whether the state change from invalid to exclusive or shared depends on whether other cores hold the same block (a simple **OR** logic on the hit signal).

1. **I**nvalid to **S**hared.  
Read miss and issue read request.  If there are other cores hold the block and the state is shared, transfer to shared state.

2. **I**nvalid to **E**xclusive.  
Read miss and issue read request.  If there are no other cores hold the block, transfer to exclusive state.

3. **I**nvalid to **M**odified.  
Write miss and issue read request.  Broadcast message to invalidate exclusive and shared state blocks.  May incur dirty write back for other modified block on other core.

4. **S**hared to **M**odified.  
Write hit.  Broadcast message to invalidate shared state blocks.

5. **S**hared to **I**nvalid.  
Snoop hit on a write.

6. **E**xclusive to **M**odified.  
Write hit.  **No need to broadcast any messages.**

7. **E**xclusive to **I**nvalid.  
Snoop hit on a write.

8. **M**odified to **S**hared.  
Snoop hit on a read.  Dirty write back and other core read the updated copy and transfer from invalid to shared.

9. **M**odified to **I**nvalid.  
Snoop hit on a write.  Dirty write back and other core read the update copy and transfer from invalid to modified. 
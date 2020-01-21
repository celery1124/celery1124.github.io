---
layout: post
title: Cache Replacement Policy Summary
categories: Architecture
tags: [cache, cache replacement]
---

# Overview

In this blog, I will summarize some non-trivial famous cache replacement policies in computer architecture.  In the future, I may organize it based on the which level of cache the replacement policy fit.

# Policies

In this section, we categroize based on the policy names.

## Belady's algorithm

Fit cache level: LLC

Belady's replacement algorithm is also known as optimal replacement policy.  The Belady's algorithm is also referred in the operating system page replacement domain.  It needs the future access information to find the replace victim (the largest re-use distance). In Lin's [Hawkeye][Back to the Future: Leveraging Belady’s Algorithm for Improved Cache Replacement] paper, they leverage the history information to implement belady's algorithm.

## RRIP (Re-Reference Interval Prediction)

Fit cache level: LLC

[RRIP][RRIP paper] first proposed the idea of re-reference interval (re-use distance) for replacement policy.  The motivation of this re-reference interval concept is that certain workloads may behave bad on LRU policy.  For example if the working set is larger than the cache associativity (thrashing access patterns).  RRIP can avoid the cache line being thrashed.  Below is the algorithm of SRRIP.

```c
Cache Hit:
(i) set RRPV of block to ‘0’
Cache Miss:
(i) search for first ‘3’ from left
(ii) if ‘3’ found go to step (v)
(iii) increment all RRPVs
(iv) goto step (i)
(v) replace block and set RRPV to ‘2’
```

## SHiP (Signature-based Hit Predictor)

Fit cache level: LLC

Note: SHiP is not limited to a specific replacement policy, but rather
can be used in conjunction with any ordered replacement policy.  In the paper, it correlates the SRRIP replacement policy to achieve great performance.

[SHiP][SHiP paper] is a predictor to better predict re-reference distance (reuse distance) rather than guide for replacement policy.  It uses a signature based method to dynamically predict reuse distance (a signature table with saturated counters like SPP).  However, it may train better results since depends on what signature it uses (in the paper, it use memory region based signature, PC based signature and program sequence based signature).  The SHiP algorithm is as follows.

``` c
if hit then 
  cache line.outcome = true;
  Increment SHCT[signature m]; 
else 
  if evicted cache line.outcome != true 
    Decrement SHCT[signature_m]; 
    cache line.outcome = false; 
    cache line.signature m = signature; 
  if SHCT[signature] == 0 
    Predict distant re-reference; 
  else 
    Predict intermediate re-reference; 
end if 

```

The innovation of this paper is using signature based method to better predict re-use distance with the cache hit/miss actions with low hardware cost which influence later works like SPP.  However, it's not able to predict the re-use distance quantitatively due to the fact that the counter only capture the hit but not the distance of the hit.

<!-- Reference -->
[Back to the Future: Leveraging Belady’s Algorithm for Improved Cache Replacement]: https://www.cs.utexas.edu/~lin/papers/isca16.pdf

[RRIP paper]:https://people.csail.mit.edu/emer/papers/2010.06.isca.rrip.pdf

[SHiP paper]:https://mrmgroup.cs.princeton.edu/papers/MICRO11_SHiP_Wu_Final.pdf
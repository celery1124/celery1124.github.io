---
layout: post
title: Cache Replacement Policy Summary
categories: Architecture
tags: [cache, cache replacement]
---

# Overview

In this blog, I will summarize some non-trivial famous cache replacement policies in computer architecture.  In the future, I may organize it based on the which level of cache the replacement policy fit.

# Policies

In this section, we categorize based on the policy names.

## Belady's algorithm

Fit cache level: LLC

Belady's replacement algorithm is also known as optimal replacement policy.  The Belady's algorithm is also referred in the operating system page replacement domain.  It needs the future access information to find the replace victim (the largest re-use distance). In Lin's [Hawkeye][Back to the Future: Leveraging Belady’s Algorithm for Improved Cache Replacement] paper, they leverage the history information to implement belady's algorithm.

## Adaptive insertion policy

Fit cache level: L2/LLC

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

## Pseudo LRU 

Fit cache level: LLC

Pseudo LRU is the same class of replacement policy as LRU.  However, it focuses on reducing the hardware complexity which maybe useful on low-cost/low-power systems. Or it's more of an **practical** implementation policy.  In a nutshell, [PseudoLRU][PseudoLRU paper] use a complete binary tree to represent the recency stack (each tree node consumes single bit which is referred to plru bit).  In contrast, for a true LRU policy you need k*log(k) bits to represent the recency stack.  Besides, in PseudoLRU insertion (promotion) it only needs to update log(k) bits compared to k*log(k) bits for true LRU.

## Dynamic set sampling for LLC

Fit cache level: LLC

Another important technique for LLC replacement policy is sampling.  [Dynamic set sampling][dynamic set sampling] samples a few dedicated sets to assess the efficacy of the desired policy.  Lots of adaptive(dynamic) policies which works like a tournament fashion which require a global view of how the hit ratio looks like can employ this approach to reduce the hardware overhead. (No need to implement on all tags).

[Sampling Dead Block Prediction][SDBP] is a perfect use case for set sampling.  The idea of SDBP is to predict dead blocks in LLC (in the paper it shows that 86% of the LLC can be dead over time).  Unlike other reference trace based predictor, SDBP only need 1.6% of the LLC access to give a pretty decent accuracy since it only sample a few numbers of sets.  Yet another practical implementation for low-power systems.

## Perceptron Learning for Reuse Prediction

Fit cache level: LLC

Hash Perceptron is yet another predictor for re-use distance prediction.  This [work][perceptron reuse prediction] comparing with SHiP and SDBP and prove to have better prediction accuracy as well as lower false positive rate.  The predictor predict re-use distance for last level cache for further insertion placement, promotion and bypass decisions. (in the future [MPPPB][Multiperspective Placement, Promotion, and Bypass] paper, the set 4 thresholds for bypassing, different LRU position placement when miss and potential promotion for hit)

The main idea of Hash Perceptron is to use multiple features (such as consecutive PCs, partial memory address, etc.) to index individual tables with saturate counters to perform the training and prediction dynamically.  The idea is not fresh new and has been already employed in branch prediction and prefetch filter in the future.


<!-- Reference -->
[Back to the Future: Leveraging Belady’s Algorithm for Improved Cache Replacement]: https://www.cs.utexas.edu/~lin/papers/isca16.pdf

[RRIP paper]:https://people.csail.mit.edu/emer/papers/2010.06.isca.rrip.pdf

[SHiP paper]:https://mrmgroup.cs.princeton.edu/papers/MICRO11_SHiP_Wu_Final.pdf

[PseudoLRU paper]:https://dl.acm.org/doi/10.1145/2540708.2540733

[dynamic set sampling]:https://ieeexplore.ieee.org/document/1635950

[SDBP]:https://dl.acm.org/doi/10.1109/MICRO.2010.24

[perceptron reuse prediction]:https://dl.acm.org/doi/10.5555/3195638.3195641

[Multiperspective Placement, Promotion, and Bypass]:https://dl.acm.org/doi/abs/10.1145/3123939.3123942
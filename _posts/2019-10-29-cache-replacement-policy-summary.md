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


<!-- Reference -->
[Back to the Future: Leveraging Belady’s Algorithm for Improved Cache Replacement]: https://www.cs.utexas.edu/~lin/papers/isca16.pdf
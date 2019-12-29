---
layout: post
title: C++ NULL pointer
categories: C++
tags: [c++, NULL, nullptr]
---

One of most important feature of C/C++ language is the low level, more precise control of the program, i.e. to manipulate the pointers.  For address 0x0 we usually refer it as NULL pointer.  In C we are familiar to use NULL which is a macro defined as below.

```c
#define NULL 0
```

However, in C++, the definition of NULL may cause problems. For example C++ support function reloads by the argument list. Consider function reloading as below.

```c
int fun(int);
int fun(int *);
```

It will be ambigus for the NULL as input. (fun(NULL) will invoke function with int as argument).  Thus, C++ introduce nullptr to replace NULL pointer in C.  Another difference between C and C++ that worth to notice is the **implicit conversion**.  C seems to have more implicit conversion which is absolutely bad for programmers (see [here](https://en.cppreference.com/w/c/language/conversion)).  However, C++ gets rid of lots of the implicit conversion (for instance, the pointer implicit cast).  So, C++ is more type safe and nullptr is one of the examples (nullptr has a dedicated type which is nullptr_t).

In a word, in C++ it's recommand to replace all NULL with nullptr.
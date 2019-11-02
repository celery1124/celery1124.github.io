---
layout: post
title: Comparison between c++11 copy and move
categories: C++
tags: [c++11, move constructor, move assignment]
---

One core feature of C++ is the introduction of constructor and destructor.  This helps the object life cycle management (we don't need to malloc/free explicitly).  One of the confusion I got is about copy and move constructor and assignemnt.  This problem raised when I was working with the [OpenCL](https://www.khronos.org/opencl) to program on a heterogeneous system.  OpenCL C++ bindings needs to manage abstract resouces (device, kernel, command queue, buffer, etc.).  The resouces usually contain some memory structure, so it needs to be careful about the memory management.

## copy vs assignment.
First we need to understand the differences between copy and assignment.  In c since we bassically using the primitive type, they are bassically the same thing.  However, if we are dealing with a class object (when it contains alloated resources), it has clear difference.  Consider a code segment like this:
```c++
class A {
  public: 
    char *buf;
    A(int size) {buf = (char *)malloc(sizeof(char)*size);}
    ~A(){free buf;}
};
A newA (A tmp) {
  return tmp;
}

int main() {
  A a1(16);
  A b1(a1);
  b1 = newA(a1);
}
```

The **copy** happens in three cases:
- new object construct with existing object as argument (explicit)
- regular fuction argument (not a reference), like the tmp local variable in newA function.
- function return, return the local variable tmp to the outer main function.

Here we can intuitively think about the **move** operation.  Passing an object with internal resources will introduing extra copy overhead.  If we can implement a move semantic, we can avoid the extra function call/return for the construtor/destructor.

## copy vs move.
The concept of copy and move are pretty straight forward, depends on whether the resouces need to be actually duplicated.  We don't need to explain the details here.  Just take a look at the code.

Note: whether the constructor or the assignment is called can be understood by the function argument.  I kind of ambigere the consturctor and assignment above, but only show the concept of copy in the code.
- copy constructor

```c++
class_name ( const class_name & )	(1)	
class_name ( const class_name & ) = default;	(2)	
class_name ( const class_name & ) = delete;	(3)	
```

- copy assignment

```c++
class_name & class_name :: operator= ( class_name )	(1)	
class_name & class_name :: operator= ( const class_name & )	(2)	
class_name & class_name :: operator= ( const class_name & ) = default;	(3)	(since C++11)
class_name & class_name :: operator= ( const class_name & ) = delete;	(4)	(since C++11)
```

- move constructor

``` c++
class_name ( class_name && )	(1)	(since C++11)
class_name ( class_name && ) = default;	(2)	(since C++11)
class_name ( class_name && ) = delete;	(3)	(since C++11)
```

- move assignment
```c++
class_name & class_name :: operator= ( class_name && )	(1)	(since C++11)
class_name & class_name :: operator= ( class_name && ) = default;	(2)	(since C++11)
class_name & class_name :: operator= ( class_name && ) = delete;	(3)	(since C++11)
```

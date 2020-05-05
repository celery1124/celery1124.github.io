---
layout: post
title: Google C++ Testing Framework
categories: C++
tags: [GTest, unit test, testing]
---

I have been noticing that lots of huge C/C++ projects use Google testing framework (i.e. GTest).  However, in the past I thought this is not worth integrating third-party testing framework in my medium/small projects.  I am used to writing my own test case for unit tests which is simple and flexible to change according to my needs and more importantly doesn't need any dependant packages.  Recently, I found most platforms already has Gtest installed and integrating Gtest to my own project can actually accelerate unit test process with a lot of advanced features (like repeat test and break on failure) provided by Gtest. This blog summarizes the basic elements of Gtest, usage and provide some useful resources.

## Overview

When using Gtest, you start by writing assertions, which are statements that check whether a condition is true. An assertion's result can be success, nonfatal failure, or fatal failure. If a fatal failure occurs, it aborts the current function; otherwise the program continues normally.  Tests use assertions to verify the tested code's behavior. If a test crashes or has a failed assertion, then it fails; otherwise it succeeds.

## Semantic

### Test Macro

GTest provide two macros to wrapper the unit test, i.e. **TEST** and **TEST_F**. 

+ **TEST**, the first argument is the name of the test suite, and the second argument is the test's name within the test suite. Both names must be valid C++ identifiers, and they should not contain any underscores (_).

+ **TEST_F** (F stands for fixtures), like TEST(), the first argument is the test suite name, but for TEST_F() this must be the name of the test fixture class. Noted TEST_F is design for more complex unit testing.  You can test with test class derived from **testing::Test**.  Below are detailed steps.

    1. Derive a class from ::testing::Test . Start its body with protected:, as we'll want to access fixture members from sub-classes.
    2. Inside the class, declare any objects you plan to use.
    3. If necessary, write a default constructor or SetUp() function to prepare the objects for each test. A common mistake is to spell SetUp() as Setup() with a small u - Use override in C++11 to make sure you spelled it correctly.
    4. If necessary, write a destructor or TearDown() function to release any resources you allocated in SetUp().
    5. If needed, define subroutines for your tests to share.

Example code can be seen in the gtest [primer].

### Run tests

After register your test cases through **TEST** and **TEST_F** Marcos.  You can invoke you tests through a main function and GTest will automatically pick up the test codes like below.

``` c++
int main(int argc, char **argv) {
  ::testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}
```

## Build

After installing the GTest in your system, (either through apt/yum or compiled through source), you can link GTest through

```bash
-lgtest -lgtest-main -pthread
```

Also, you can build through more modern CMake. See [here].

## Reference
[https://developer.ibm.com/articles/au-googletestingframework/](https://developer.ibm.com/articles/au-googletestingframework/)

[https://github.com/google/googletest/blob/master/googletest/docs/primer.md](https://github.com/google/googletest/blob/master/googletest/docs/primer.md)

[https://github.com/google/googletest](https://github.com/google/googletest)

[primer]: https://github.com/google/googletest/blob/master/googletest/docs/primer.md
[here]: https://notes.eatonphil.com/unit-testing-c-code-with-gtest.html
# ISO C++ standard proposal for supporting SPMD programming in the core language

## Table of contents

    1. Introduction
    2. Background and Motivation
    3. Language Additions
    4. Discussion
    5. References

## 1. Introduction

NOTE: This proposal struction was taken directly from the [Call for Library
Proposals](http://open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3370.html)

SPMD programming should be in C++ as it expresses data parallelism in the type
system...where it should be.

## 2. Background and Motivation

Parallelism has been the cornerstone of compute performance on modern hardware.
Multicore CPUs, GPUs, and FPGAs all rely on expressed parallelism to be fully
utilized.  There are two forms of parallelism which C++ programmers are able to
exploit to improve performance: task parallelism and data parallelism. Task
parallelism is the execution of multiple units of work in parallel, where data
parallelism is the execution of a single task on multiple units of data in
parallel. With these two concepts, programmers then must use C++ features and
libraries to map their applications' parallelism to the correct points for
parallel hardware to quickly and correctly execute in parallel.

There are multiple methods for which an application can be written to be
executed in parallel: distributed applications, multithreading, and
vectorization. Given these options, C++ programs can potentially use all of
these methods simultaneously. An application distributed on multiple machines
can run code on multiple threads which are implemented with vector instructions.

The current state of C++ since C++11 is that programmers only have ways of
using multiple threads through the standard library: distributed applications
and vectorization are either unavailable without 3rd party libraries or
implemented as unreliable, non-portable, opaque, and confusing compiler
transformations. This proposal seeks to address the problem of vectorization
by inserting core language features into C++ which express data parallelism
through Single Program Multiple Data (SPMD) constructs.

### The Status Quo (Part 1): Vector Intrinsics

There are two levels of abstraction which programmers can use to vectorize
code: code annotations and compiler intrinsics. Compiler intrinsics are
a fairly straightforward concept to understand, but are very difficult to use.
Intrinsics simply wrap a specific hardware instruction in a callable function.
The biggest advantage here is _control_: the programmer can be very exact with
which instructions will implement a function. The drawbacks, however, make it a
less popular method.

A big barrier for most programmers against using intrinsics is their massive
deviation in syntax from how regular C++ looks as it forces users to use function
calls in place of expressions. Consider the following C++ loop, where a and b are
arrays of size 8:

```C++
for (int i = 0; i < 8; ++i) {
  b[i] = 5 * a[i];
}
```

This uses normal multiplication syntax on each element in the array and stores it
in a new resulting array. On x86 with AVX instructions, you can "unroll" the loop
and execute the multiplication in 1 single instruction instead of 8:

```C++
TODO: write the above code in AVX
```

From the point of view of syntax, there isn't any resemblence of the first to the
second. What was lost in code expression was returned in control. In this regard,
intrinsics are much like inline assembly. Instead of trusting that the compiler
will generate for the expected instrutions given input C++ code, the programmer
can instead insert _exact_ instructions.

An even more dire drawback to losing C++ expressions is loss of portability. Every
time an alias for a platform specific instruction is used, the code can no longer
be used to generate instructions for other instruction sets.

### The Status Quo (Part 2): Code Annotations

Another, more popular, avenue for vectorizing code is to use code annotations.
In C++, these come in the form of pragma statments which are intended to
communicate extra information to the compiler about structured blocks of code,
typically for-loops. Consider the following code vectorized with OpenMP:

```C++
void saxpy_omp(float a, float *x, float *y, float *out, int n)
{
# pragma omp for simd
  for (int i = 0; i < n; ++i) {
    out[i] = a * x[i] + y[i];
  }
}
```

This code is easy to understand as it is very simple in its nature: it's the
ubiquitous SAXPY example every vectorization method uses as its "hello world".
Where this style starts to break down is in more non-trivial code.
Specifically, branching and function calls are difficult to get correct without
the compiler giving up on vectorizing. For example, consider the following
OpenMP code:

```C++
TODO: write a much crazier for-loop that OpenMP will struggle to vectorize
```

There are too many questions here that the compiler has to answer about what
the programmer _really_ means. Is the function being called a vectorized
function?  Is it guarenteeed that branching will be coherent between loop
iterations? What about data alignment?

There are further issues with this as OpenMP is really built for C, meaning
that the C++ type system can very quickly push OpenMP's ability to vectorize.
One such example of this is data layout of structures.

## 3. Solution: Add SPMD to C++

## 4. Discussion

## 5. References

TODO

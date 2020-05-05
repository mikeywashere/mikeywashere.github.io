# Performance and Memory Access #
## The Problem ##

When you are working with large data sets you should understand how data is laid out in memory and how your data will be accessed.

Let's take a look at some example code to help us understand how memory access patterns can change the performance of our software.

Our example is simple matrix multiplication - see this [Wikipedia Link about Matrix Multiplication](https://en.wikipedia.org/wiki/Matrix_multiplication).

A small 10 x 10 matrix filled with 8 byte doubles will only take 800 bytes. Unless you are working with 10,000 you may not be worried about your memory access patterns.

It has been said that "Premature optimization is the enemy". A lot of time can be wasted 'solving' the wrong problem.

Let's get back to our matrix multiplication problem. Think about the memory footprint of two 1000 x 1000 matrices, or three if you want your answers in another matrix. 1,000 * 1,000 = 1,000,000 * 8 = 8,000,000. So each one takes 8 megabytes in memory. That's not a lot if memory but your CPU doesn't access memory directly, it accesses memory through the cache. Since the cache is much smaller than main memory data has to be swapped in and out, also known as mapping. The smaller your data set the less swapping that will happen. The larger your data set the more potential you have for invalidating the cache and causing it to have to load data from main memory repeatedly.

I've posted the example code at [Matrix Multiply](https://github.com/mikeywashere/Matrix-Multiply)

Since the matrices are large and we are multiplying two matrices and placing the results into another we will be accessing about 24mb of memory. So this is a nice way to show that when an algorithm needs to access larges amounts of memory it should try to do so in an orderly non-random manner when possible.

You can read here for more information on cpu/memory cache: [CPU Cache](https://en.wikipedia.org/wiki/CPU_cache)

### Accessing Data ###
Sequential access is the fastest way to access data. The CPU is faster than RAM, If we just coupled our speedy CPUs to our slow ram we'd have a CPU that spent most of its time waiting to get something to work on.

This is why CPUs have cache memory. Cache memory sits between our CPU and the RAM. Random access memory has close to equal access speeds to any location in memory, The issue is that to keep the CPU working we need to supply it with new instructions and new data all of the time or it will stall, meaning that it cannot work until data is available.

I'm going to be talking about the x86 architecture, I'm not going to pretend I know anything about any other architecture. I know there are other caching architectures but I don't know enough about them to intelligently discuss them nor do I have hardware to run any kind of test on.

So with that said...

On the x86 architecture the Level 1 cache (referred to as L1) is split into "cache lines". A common cache line size is 64 bytes. When memory is accessed that is not in the cache a cache line worth of data is fetched.

### Cache Lines ###

A cache line is 64 bytes of data. During memory access a cache line is filled - single bytes, words, etc are not read, cache lines are read.

### A *Dirty* Cache Line ###

If that cache line contains data that is "dirty" (not yet written to main memory) then the first thing to that needs to happen is writing that data to main memory. That takes time. Once that is complete then the cache line needs to be filled with the data from the main memory that was originally requested.

## Cache Mapping ##

---
#### Note ####
My personal workstation is a dual processor [Xeon E5-2687w](https://ark.intel.com/content/www/us/en/ark/products/64582/intel-xeon-processor-e5-2687w-20m-cache-3-10-ghz-8-00-gt-s-intel-qpi.html) - Each processor contains 20 MB Intel® Smart Cache and 8 cores

---

### CoreInfo.exe ###

Using CoreInfo.exe from Microsoft to see the cache "layout". Download here - [live](https://live.sysinternals.com/) and [docs](https://docs.microsoft.com/en-us/sysinternals/) here.

```
Coreinfo v3.5 - Dump information on system CPU and memory topology
Copyright (C) 2008-2020 Mark Russinovich
Sysinternals - www.sysinternals.com

Logical Processor to Cache Map:
**------------------------------  Data Cache          0, Level 1,   32 KB, Assoc   8, LineSize  64
**------------------------------  Instruction Cache   0, Level 1,   32 KB, Assoc   8, LineSize  64
**------------------------------  Unified Cache       0, Level 2,  256 KB, Assoc   8, LineSize  64
****************----------------  Unified Cache       1, Level 3,   20 MB, Assoc  20, LineSize  64
--**----------------------------  Data Cache          1, Level 1,   32 KB, Assoc   8, LineSize  64
--**----------------------------  Instruction Cache   1, Level 1,   32 KB, Assoc   8, LineSize  64
--**----------------------------  Unified Cache       2, Level 2,  256 KB, Assoc   8, LineSize  64
----**--------------------------  Data Cache          2, Level 1,   32 KB, Assoc   8, LineSize  64
----**--------------------------  Instruction Cache   2, Level 1,   32 KB, Assoc   8, LineSize  64
----**--------------------------  Unified Cache       3, Level 2,  256 KB, Assoc   8, LineSize  64
------**------------------------  Data Cache          3, Level 1,   32 KB, Assoc   8, LineSize  64
------**------------------------  Instruction Cache   3, Level 1,   32 KB, Assoc   8, LineSize  64
------**------------------------  Unified Cache       4, Level 2,  256 KB, Assoc   8, LineSize  64
--------**----------------------  Data Cache          4, Level 1,   32 KB, Assoc   8, LineSize  64
--------**----------------------  Instruction Cache   4, Level 1,   32 KB, Assoc   8, LineSize  64
--------**----------------------  Unified Cache       5, Level 2,  256 KB, Assoc   8, LineSize  64
----------**--------------------  Data Cache          5, Level 1,   32 KB, Assoc   8, LineSize  64
----------**--------------------  Instruction Cache   5, Level 1,   32 KB, Assoc   8, LineSize  64
----------**--------------------  Unified Cache       6, Level 2,  256 KB, Assoc   8, LineSize  64
------------**------------------  Data Cache          6, Level 1,   32 KB, Assoc   8, LineSize  64
------------**------------------  Instruction Cache   6, Level 1,   32 KB, Assoc   8, LineSize  64
------------**------------------  Unified Cache       7, Level 2,  256 KB, Assoc   8, LineSize  64
--------------**----------------  Data Cache          7, Level 1,   32 KB, Assoc   8, LineSize  64
--------------**----------------  Instruction Cache   7, Level 1,   32 KB, Assoc   8, LineSize  64
--------------**----------------  Unified Cache       8, Level 2,  256 KB, Assoc   8, LineSize  64
----------------**--------------  Data Cache          8, Level 1,   32 KB, Assoc   8, LineSize  64
----------------**--------------  Instruction Cache   8, Level 1,   32 KB, Assoc   8, LineSize  64
----------------**--------------  Unified Cache       9, Level 2,  256 KB, Assoc   8, LineSize  64
----------------****************  Unified Cache      10, Level 3,   20 MB, Assoc  20, LineSize  64
------------------**------------  Data Cache          9, Level 1,   32 KB, Assoc   8, LineSize  64
------------------**------------  Instruction Cache   9, Level 1,   32 KB, Assoc   8, LineSize  64
------------------**------------  Unified Cache      11, Level 2,  256 KB, Assoc   8, LineSize  64
--------------------**----------  Data Cache         10, Level 1,   32 KB, Assoc   8, LineSize  64
--------------------**----------  Instruction Cache  10, Level 1,   32 KB, Assoc   8, LineSize  64
--------------------**----------  Unified Cache      12, Level 2,  256 KB, Assoc   8, LineSize  64
----------------------**--------  Data Cache         11, Level 1,   32 KB, Assoc   8, LineSize  64
----------------------**--------  Instruction Cache  11, Level 1,   32 KB, Assoc   8, LineSize  64
----------------------**--------  Unified Cache      13, Level 2,  256 KB, Assoc   8, LineSize  64
------------------------**------  Data Cache         12, Level 1,   32 KB, Assoc   8, LineSize  64
------------------------**------  Instruction Cache  12, Level 1,   32 KB, Assoc   8, LineSize  64
------------------------**------  Unified Cache      14, Level 2,  256 KB, Assoc   8, LineSize  64
--------------------------**----  Data Cache         13, Level 1,   32 KB, Assoc   8, LineSize  64
--------------------------**----  Instruction Cache  13, Level 1,   32 KB, Assoc   8, LineSize  64
--------------------------**----  Unified Cache      15, Level 2,  256 KB, Assoc   8, LineSize  64
----------------------------**--  Data Cache         14, Level 1,   32 KB, Assoc   8, LineSize  64
----------------------------**--  Instruction Cache  14, Level 1,   32 KB, Assoc   8, LineSize  64
----------------------------**--  Unified Cache      16, Level 2,  256 KB, Assoc   8, LineSize  64
------------------------------**  Data Cache         15, Level 1,   32 KB, Assoc   8, LineSize  64
------------------------------**  Instruction Cache  15, Level 1,   32 KB, Assoc   8, LineSize  64
------------------------------**  Unified Cache      17, Level 2,  256 KB, Assoc   8, LineSize  64

```

If memory is accessed sequentially then the cache lines are filled and written using a predictable, sequential method. If our memory access is random then a cache line might be filled on one instruction, flushed, then filled again a few instructions later. With a random pattern of access we could force the CPU to stall while waiting for data. And then stall again while waiting for that same data. Each stall isn't a terrific waste of time if the CPU has something else to work on, but if our code forces stall after stall we will definitely see a performance impact.

The Code
In the project that I've posted the example code at [Matrix Multiply](https://github.com/mikeywashere/Matrix-Multiply) you will find four files of interest.

The interface IMatrix that describes how to access a basic matrix. An indexer with an x and a y position and two properties Rows and Columns that tell how large the matrix is.


MatrixBase witch implements a few basic methods. MatrixMultiply, RandomFill and ToString along with basic data access. This class also implements the data storage as a single dimensional array which lets us control the access order, row or column optimized. Row optimized accesses the memory sequentially if the row index is incremented by one, column optimized accessed the data sequentially

RowOptimizedMatrix which is basically just redefining MatrixBase because it is already accessing the data in a row optimized way.

ColumnOptimizedMatrix which changes the data access to be in a column optimized way.

The Test
The basis of the test is this data:

``` csharp
const int repeat = 3;
const int size = 1000;

static MatrixBase co1 = new ColumnOptimizedMatrix(size, size);
static MatrixBase ro1 = new RowOptimizedMatrix(size, size);
static MatrixBase co2 = new ColumnOptimizedMatrix(size, size);
static MatrixBase ro2 = new RowOptimizedMatrix(size, size);
static MatrixBase aco1 = new ColumnOptimizedMatrix(size, size);
static MatrixBase aro1 = new RowOptimizedMatrix(size, size);
```

You will find in the matrix multiply code that there are a few parallel options defined.

``` csharp
var parallelOptions1 = new ParallelOptions
{
    MaxDegreeOfParallelism = Environment.ProcessorCount * 2
};

var parallelOptions2 = new ParallelOptions
{
    MaxDegreeOfParallelism = Environment.ProcessorCount * 4
};
```

I found that using option 2 on my machine yielded the best results. You mileage may vary.

In program.cs there are tests for optimized and several non-optimized scenarios.

The Results (I'll be rerunning these tests soon as I have more cores now)
The winner is "Optimized input and column output" although "Optimized input and row output" was a very close second

```
Running: Optimized input and column output
ms: 3,427.00
ms: 3,238.00
ms: 3,184.00

Optimized input and column output: 3,283.00

Running: Optimized input and row output
ms: 3,320.00
ms: 3,565.00
ms: 3,260.00

Optimized input and row output: 3,381.67

Running: Non-Optimized, input 2 matrices by column and column output
ms: 6,920.00
ms: 6,829.00
ms: 6,799.00

Non-Optimized, input 2 matrices by column and column output: 6,849.33

Running: Non-Optimized, input 2 matrices by column and row output
ms: 6,946.00
ms: 7,014.00
ms: 6,671.00

Non-Optimized, input 2 matrices by column and row output: 6,877.00

Running: Non-Optimized, input 2 matrices by row and column output
ms: 6,981.00
ms: 7,174.00
ms: 7,283.00

Non-Optimized, input 2 matrices by row and column output: 7,146.00

Running: Non-Optimized, input 2 matrices by row and row output
ms: 7,288.00
ms: 7,656.00
ms: 7,345.00

Non-Optimized, input 2 matrices by row and row output: 7,429.67
```

Why
The matrix multiply method accesses memory from one matrix in a row ordered method, and from a second matrix in a column ordered method. By creating data structures that are optimized for their usage scenario you can see a performance gain of up to two times (on my system).

Being twice as fast at anything is good. Understanding and being able to prove why one piece of code is faster than another is very good.
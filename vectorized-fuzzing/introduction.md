---
description: Basic introduction of the Vectorized emulation note from Brandon Falk
---

# Introduction

## The Ultimate Goal&#x20;

* Take standard applications and JIT them to their AVX-512 equivalent, such that we can fuzz 16 VMs at a time per thread.
* The net result of this work allows for high-performance fuzzing (\~ 40 billion to 120 billion instructions per second), depending on the target.
* While gathering differential coverage on code, register, and memory state.
* By gathering more than just code coverage, we can track the state of code deeper than just code coverage itself&#x20;

### Notes for me

* MIPS Emulator development&#x20;
* JIT internals&#x20;

## Short Intro to Snapshot Fuzzing&#x20;

Snapshot fuzzing is a method of fuzzing where you start from a partially executed system state.&#x20;

If you had prior experience with user-mode fuzzing, you would say, "Why can't you just kill the process and rerun it?". What if you are fuzzing the kernel and OS? You will have encountered BSOD (Blue Screen Of Death) causing the system to crash. So we have a snapshot. A snapshot is basically when you dump memory and register the state to a core dump. Having a full memory and register state is the beginning of fuzzing. You can dump to any emulator, set up memory contents and permissions, set up register state, and continue execution. (minidump on Windows). Usually, when you are going for paper and research, you would have to automate and prove that your fuzzer was faster than some other fuzzer and how exotic it is. &#x20;

Having a memory state and register state means that we can inject and modify file contents in memory and continue execution with new input.&#x20;

Depending on the target, this can be difficult. But usually, you can make some custom rigging using, `strace` which is a debugging tool that can be used to discover the system resources in use by a workload. If your target is Windows, you can try using Logger/LogViewer which is implemented in [Debugging Tools for Windows](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/), DTrace, and Process Monitor to check FDs (File Descriptors).&#x20;

Time is significant when you are fuzzing, and so is the bug you find too&#x20;

### Vector Instruction Set&#x20;

A vector instruction is executed by a vector unit, which is similar to a conventional Single Instruction Multiple Data (SIMD) instruction. Each vector instruction can perform the same type of operation on multiple samples (for us, it's VM). The instruction can directly be run on the data in the OB without loading the data into the vector register with a data loading instruction. The data types supported are FP16, FP32, and INT32. The vector instruction supports recursive execution and the direct operation of vectors that are not stored in continuous memory space.&#x20;

### SIMD introduction&#x20;

**SIMD** stands for **S**ingle **I**nstruction and **M**ultiple **D**ata. Which is also used in any 64-bit x86 system `memcpy()` implementation. It's a computing method that enables the processing of multiple data with a single instruction.&#x20;

For example, adding two arrays: C\[i] = A\[i] + B\[i] for the entire dataset. A simple solution is to iterate over each pair of elements and perform the addition sequentially.

&#x20;_"SIMD operations are commonly used in C/C++,  and in certain cases, the compiler can automatically vectorize the code. The GCC and Clang compilers provide `vector_size` and `ext_vector_type` attribute to auto-vectorize C/C++ code that has data parallelism."  **Boost JavaScript performance by Exploiting Vectorization using SIMD.js**_&#x20;

### SSE introduction&#x20;

It's the intel Streaming SIMD Extensions (SSE) comprise a set of extensions to the Intel x86 architecture that is designed to greatly enhance the performance of advanced media and communication applications.&#x20;

SSE provides 128-bit SIMD operations for x86. SSE introduced 16 128-bit registers named xmm0 through xmm15. These 128-bit registers can be treated as groups of different-sized, smaller pieces of data that sum up to 128 bits.

* 4 single precision floats
* 2 double precision floats
* 2 64-bit integers&#x20;
* 4 32-bit integers&#x20;
* 8 16-bit integers&#x20;
* 16 8-bit integers&#x20;

### Differences Between Intel AVX-512 and Intel AVX10&#x20;

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>Table 1-3, Feature Differences Between Intel AVX-512 and Intel AVX10 </p></figcaption></figure>

Let me explain the difference between these new words.

### Intel AVX&#x20;

* 128/256 bit Floating Point&#x20;
* 16 registers&#x20;
* NDS (and AVX128)&#x20;
* Improved blend&#x20;
* MASKMOV: Stores selected bytes from the source operand (first operand) into a 64-bit memory location. The mask operand (second operand) selects the source operand and is written to memory.&#x20;
* Implicit unaligned: Each data element is mapped to the next byte boundary



### Future-building Xeon Phi stuff&#x20;

* [Intel Xeon PHI 7210 ](https://www.ebay.com/itm/364207471622?chn=ps\&mkevt=1\&mkcid=28\&srsltid=AfmBOopFKo4hYKKt0i5VUNthjfXnvJVwBT3tqu0ybmCYRW7ifbC2czNY4HY\&autorefresh=true)
* [Intel Node s7200AP Motherboard ](https://www.ebay.com/itm/175459054222?chn=ps\&mkevt=1\&mkcid=28\&srsltid=AfmBOoohJXByD2O\_T4sNN\_D3dslds1MmITaKuzZw8q4M3CJVjTgRY3urObM\&com\_cvv=81c269aab9bc5fc4177fabac3c77acd26f491510dd5f94a06570d6c819846581)










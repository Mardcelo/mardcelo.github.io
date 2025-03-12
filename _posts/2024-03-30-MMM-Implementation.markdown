---
layout: post
title:  "MMM - Matrix-Matrix Multiplication: Experiment"
date:   2024-03-30 02:10:10 +0100
categories: Algorithm, Optimization 
usemathjax: true
---

### Introduction

Matrix-Matrix Multiplication (MMM) is one for the important numerical kernel functionality, used in various places such areas as Network, Linear Systems of Equation, Transformation of Co-ordinate Systems, and population modeling. 

I was trying to test some stuffs out since I just started learning optimizations, and also high performance programming after getting Intel Xeon Phi. The codes from this slides was rather taken in some lecture that I have learnt, or made by myself while working on optimization.  

---

#### Floating Point Operations 

Floating point operations are mathemtical operations that involve floating point numbers. These operations are required for signed calculation and are used in embedded applicaitons. They are IEEE Standard 754 for Binary Floating-Point Arithmetic. 

It is important attribute in developing the flaoting-point algorithms. 

---

#### Big O notation

Big O (Big-Oh) notation is a mathematical notation that is about finding an asymptotic upper bound. Why this is important? Because, this is also a way to measure how fast is your algorithm. 

This can be shown as: 

$$O(f(n))$$ 

**Where:** 

  $n$ is the input size 
  
  $f$ is a function 
  
---

### Equation 

$$C_{ij} = \sum\limits_{k = 1}^m {a_{i,k}\ b_{k,j}}$$

Given a $k$ x $m$ matrix $A = [a_{i,j}]$ and an $m$ x $n$ matrix $B = [b_{i,j}]$ the product $C = AB$ is a $k$ x $m$ matrix with entries. 

---

### Example 

So assume if you have $2$ by $3$, $3$ by $2$ matrices 


$$\begin{pmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23}\end{pmatrix}\begin{pmatrix}b_{11} & b_{12} \\ b_{21} & b_{22} \\ b_{31} & b_{32}\end{pmatrix}$$ 


The end product should look like $C_{21}$ = $a_{21}\ b_{11}$ + $a_{22}\ b_{21}$ + $a_{23}\ b_{31}$  

$C_{21}$ the subscript $2$ and $1$ should represent $i$ and $j$ from the equation. 

But for the application in real life, usually $C$ = $C\+\ AB$ is implemented instead of $C$ $=\ AB$. 

Given that two $n$ x $n$ matrices $A$, $B$. 

MMM computed as $C$ = $C\+\ AB$ requires $n^3$ additions for total of $2n^3$ = $O(n^3)$ floating point operations. 

Since, the matrices have size of $O(n^2)$. The reuse is given by $O({n^3 \over n^2})$, which is $O(n)$.

---

### Experiment 

This experiment is to check the knowledge in MMM Multiplication that I have learnt during the courses I took. 

---


### **MMM.c** ###

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/time.h>

#define n 1024
double A[n][n];
double B[n][n];
double C[n][n];

float tdiff(struct timeval *start,
            struct timeval *end) {
  return (end->tv_sec - start->tv_sec) +
    1e-6*(end->tv_usec - start->tv_usec);
}

int main(int argc, const char *argv[]) {
  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < n; ++j) {
      A[i][j] = (double)rand() / (double)RAND_MAX;
      B[i][j] = (double)rand() / (double)RAND_MAX;
      C[i][j] = 0;
    }
  }

  struct timeval start, end;
  gettimeofday(&start, NULL)

  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < n; ++j) {
      for (int k = 0; k < n; ++k) {
        C[i][j] += A[i][k] * B[k][j];
      }
    }
  }

  gettimeofday(&end, NULL);
  printf("%0.6f seconds\n", time_difference(&start, &end));
  return 0;
}
```

Code might be messy, but it does the work 

This code performs martrix multiplication of two n x n matrices A and B, and stores the result in matrix C. Where, the size of the matrices is defined as a constant n = 1024. 

The code intilizes the matrices A, B, and C by filling them with random values from 0 and 1 using the `rand()` function from the `<stdlib.h>`. 

And then, the code measures the time it takes to perform the matrix multiplication using the `gettimeofday()` funciton from the `<sys/time.h>` library. 

The start time is recoded before the multiplication, and the end time is recorded after the multiplication has finished. And then the elapsed time is then calculated by subtracting the start time and the end time using the `tdiff()` function (time difference). 

The matrix multiplication is performed using three nested for loops. 
The outer two loops iterate over the rows and columns of matrix C, while the inner loop iterates over the columns of matrix A and the rows of matrix B. 

The value of each element of matrix C is calculated as the sum of the products of corresponding elements in the rows of matrix A and columns of matrix B. 

After all of these painful jobs, they elapsed time is printed to the console using the `printf()` function. 

---
### How to Measure Runtime? 

Like what we used `gettimeofday()`, there is few options that have different usage and you can choose from them for what purpose you are using. Since, you need to measure only what you want to measure, and proper machine state. 

1.  `clock()` in C 
    - Process specific, low resolution, very portable.

2. `gettimeofday()`
     - Measures wall clock tim , higher resolution, somewhat portable.

3. Performance counter (e.g, TSC on Intel)
     - Measures cycles (also wall clock time), highest resolution, not portable.
     - Problematic with frequency scaling
   
NOTE: Check how reproducible; if not repducible: fix it. 

---
## Sources/ Special Thanks 

[Prof. Püschel]([url](https://acl.inf.ethz.ch/people/markusp/)) -> Thank you for the Amazing course ASL (Advanced System Lab)!  

[Prof. Leiserson]([url](https://ocw.mit.edu/search/?q=Prof.+Charles+Leiserson)) -> Thank you for the Amazing course 6.172 (Performance Engineering Of Software Systems)!

[Agner Fog Ph.D.  ]([url](https://www.agner.org/contact/?e=0#0)) -> Thank you for your dedication and hard work in Optimization field! 


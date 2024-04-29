---
layout: post
title:  "MMM - Matrix-Matrix Multiplication: Experiment"
date:   2024-03-30 02:10:10 +0100
categories: Algorithm 
usemathjax: true
---
### Introduction

Matrix-Matrix Multiplication (MMM) is one of the important numerical kernel functionality, used in various places such areas as Network, Linear Systems of Equation, Transformation of Co-ordinate Systems, and population modeling.  

#### Floating Point Operations 

#### Big O notation
Big O (Big-Oh) notation is a mathematical notation that is about finding an asymptotic upper bound.

This can be shown as: 

$$O(f(n))$$ 

Where: 

  $n$ is the input size 
  
  $f$ is a function 
  
### Equation 

$$C_{ij} = \sum\limits_{k = 1}^m {a_{i,k}b_{k,j}}$$

Given a $k$ x $m$ matrix $A = [a_{i,j}]$ and an $m$ x $n$ matrix $B = [b_{i,j}]$ the product $C = AB$ is a $k$ x $m$ matrix with entries. 

So assume if you have $2$ by $3$, $3$ by $2$ matrices 

$$\begin{pmatrix}
a_{11} & a_{12} & a_{13}\\
a_{21} & a_{22} & a_{23} 
\end{pmatrix}$$    

$$\begin{bmatrix} a_{1,1} & a_{1,2} \\ a_{2,1} & a_{2,2}\end{bmatrix}\begin{bmatrix}b_{1,1} & b_{1,2} \\ b_{2,1} & b_{2,2}\end{bmatrix} \boldsymbol{\neq} \begin{bmatrix}a_{1,1}b_{1,1} & a_{1,2}b_{1,2} \\ a_{2,1}b_{2,1} & a_{2,2}b_{2,2}\end{bmatrix}$$


The end product should look like $C_{21}$ = $a_{21}\ b_{11}$ + $a_{22}\ b_{21}$ + $a_{23}\ b_{31}$  

$C_{21}$ the subscript $2$ and $1$ should represent $i$ and $j$ from the equation. 

But for the application in real life, usually $C$ = $C\+\ AB$ is implemented instead of $C$ $=\ AB$. 

Given that two $n$ x $n$ matrices $A$, $B$. 
MMM computed as $C$ = $C\+\ AB$ requires $n^3$ additions for total of $2n^3$ = $O(n^3)$ floating point operations. 
Since the matrices have size of $O(n^2)$, the reuse is given by $O({n^3 \over n^2})$ which is $O(n)$.

### Strassen's Algorithm 




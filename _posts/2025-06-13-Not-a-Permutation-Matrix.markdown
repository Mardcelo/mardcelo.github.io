---
layout: post
title:  "GreyCTF Not a Permutation Matrix"
date:   2025-02-27 02:10:10 +0100
categories: CTF 
usemathjax: true
redirect_to:
  - https://project.mard.kr/s/D4plFGTaw
---

# [GreyCat CTF] Not a Permutation Matrix 

Mard (Rick) Kim
- [TOC]

## Challenge Description

> **Not A Permutation Matrix - 1000 points**
> Category: Crypto
> 
>**Description:**
> I got a permute, I got a matrix, permute matrix.

<details>
    <summary>Challenge File</summary>
    
```sage=
import random
from base64 import b64encode, b64decode
from functools import reduce

SEED = int(0xcaffe *3* 0xc0ffee)

# https://people.maths.bris.ac.uk/~matyd/GroupNames/61/Q8s5D4.html
G = PermutationGroup([
    [(1,2,3,4),(5,6,7,8),(9,10,11,12),(13,14,15,16),(17,18,19,20),(21,22,23,24),(25,26,27,28),(29,30,31,32)],
    [(1,24,3,22),(2,23,4,21),(5,14,7,16),(6,13,8,15),(9,27,11,25),(10,26,12,28),(17,31,19,29),(18,30,20,32)],
    [(1,5,12,30),(2,6,9,31),(3,7,10,32),(4,8,11,29),(13,25,19,21),(14,26,20,22),(15,27,17,23),(16,28,18,24)],
    [(1,26),(2,27),(3,28),(4,25),(5,14),(6,15),(7,16),(8,13),(9,23),(10,24),(11,21),(12,22),(17,31),(18,32),(19,29),(20,30)]
])
num2e = [*G]
e2num = {y:x for x,y in enumerate(num2e)}
b64encode_map = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0987654321+/"
b64decode_map = {c:i for i,c in enumerate(b64encode_map)}

def gen_random_mat(seed: int, size: int):
    random.seed(seed)
    return matrix(size, size, random.choices([0,1], k=size*size))

def mat_vec_mul(mat, vec):
    return [reduce(lambda x,y: x*y, (a^b for a,b in zip(vec, m))) for m in mat]

def hash_msg(msg: bytes):
    emsg = b64encode(msg).decode().strip("=")
    sz = len(emsg)
    vec = [num2e[b64decode_map[c]] for c in emsg]
    mat = gen_random_mat(SEED, sz)
    for _ in range(10):
        # Since G is non-abelian, this is a pretty good hash function!
        vec = mat_vec_mul(mat, vec)
    return ''.join((b64encode_map[e2num[c]] for c in vec))

if __name__ == "__main__":
    from flag import flag # secret!!
    import re

    assert re.match(r"^grey\{.+\}$", flag.decode())
    ct = hash_msg(flag)
    print(ct)

# Program output:
# aO3qDbHFoittWTN6MoUYw9CZiC9jdfftFGw1ipES89ugOwk2xCUzDpPdpBWtBP3yarjNOPLXjrMODD
```

</details>

## Introduction to the Permutation Group

This challenge used a specific group denoted with 
$$Q_8 \rtimes_5 D_4$$ 

which is a p-group, metabelian, nilpotent (class 2), monomial. 

You would say huh?? so did I. 

The basic meaning of this is that $Q_8 \rtimes_5 D_4$ means a semidirect product ($\rtimes$) of two groups: $Q_8$ (the quaternion group of order $8$) and $D_4$ (the Dhihedral group of order $8$). The $5$ subscript indicates a particular way these groups are combined. Which it signifies a specific action of $D_4$ on $Q_8$ that determines how the semidirect product is formed. It's the 1st semidriect product out of several possible ways to combine $Q_8$ and $D_4$. This was found in this [link](https://people.maths.bris.ac.uk/~matyd/GroupNames/1/C4sC4.html), which was provided in our source code. 
 

## Analysis of the Challenge Script

Going back to our challenge, we were given a Sage script that included a function called `hash_msg`. This function took an input, base64-encoded it, and converted the characters into elements of a specific permutation group `G`. It then iteratively applied a matrix-vector-like multiplication using group operations. Finally, the resulting group elements were converted back into a base64 string.

The `hash_msg` function is built upon a non-abelian group `G`. Where we got this from the comment of the code "Since G is non-abelian, this is a pretty good hash function!" 

```python=
def hash_msg(msg: bytes):
    emsg = b64encode(msg).decode().strip("=")
    sz = len(emsg)
    vec = [num2e[b64decode_map[c]] for c in emsg]
    mat = gen_random_mat(SEED, sz)
    for _ in range(10):
        # Since G is non-abelian, this is a pretty good hash function!
        vec = mat_vec_mul(mat, vec)
    return ''.join((b64encode_map[e2num[c]] for c in vec))
```

This might be true, that non-abelian operations can be difficult to reverse. But there is a plot twist. 

While `G` itself is non-ablian, we can consider its structure "modulo its center." The center of a group, $Z(G)$. It consists of all elements that commute with every element in $G$. To understand the trick, we need to look at the quotient group $Q = G \backslash Z(G)$.

For the specific group `G` used in this challenge $Q_8 \rtimes_5 D_4$ is isomorphic to $C_2 \times C_2$ (the Klein four-group, or $\mathbb{F}_2^2$ as a vector space). More importantly, the quotient group $Q=G \backslash Z(G)$ turns out to be isomorphic to $(C_2)^4$ (the elementary abelian group of order 16, or $\mathbb{F}_2^4$ as a 4-dimensional vector space over the field of two elements, $\mathbb{F}^2$). This is good news for us because $\mathbb{F}_2^4$ is abelian, and its operations are simple vector additions (XORs).

```sage
sage: ZG = G.center()
sage: ZG
Group([ (1,17)(2,18)(3,19)(4,20)(5,21)(6,22)(7,23)(8,24)(9,25)(10,26)(11,27)(12,28)(13,29)(14,30)(15,31)(16,32), (1,22)(2,21)(3,24)(4,23)(5,17)(6,18)(7,19)(8,20)(9,30)(10,29)(11,32)(12,31)(13,25)(14,26)(15,27)(16,28) ])
sage: ZG.order()
4
sage: ZG.structure_description()
'C2 x C2'
```

The center $Z(G)$ is isomorphic to $C_2 \times C_2$, the Klein four-group. 

If we look into quotient group $Q = G \backslash Z(G)$:

```sage
sage: Q, phi_G_to_Q = G.quotient(ZG, "homomorphism")  |-> g*ZG
sage: Q.order()
16
sage: Q.is_abelian()
True
sage: Q.structure_description()
'C2 x C2 x C2 x C2'
```

The quotient group $Q = G \backslash Z(G)$ has order of $64 \backslash 4$ = 16. Crucially, it is abelian and isomorphic to $(C_2)^4$, which can be viewed as a 4-dimensional vector space over the field $\mathbb{F}_2$. Operations modulo the center are simple XOR operations. 


## Formulating Our Solver 

The challenge asks us to reverse the `hash_msg` function. The core idea is to leverage the structure of the group $G$ to break down the non-abelian problem into manageable, linear subproblems. 

### 1. Group-Based Transformation

Each round of the `mat_vec_mul` function applies the following transformation:
If $V_{\text{in}}$ is the input vector of group elements and $V_{\text{out}}$ is the output vector, then for each element $i$ is

$$
V_{\text{out}}[i] = \prod_{j \mid B_{ij} = 1} V_{\text{in}}[j]
$$

Let:
*   $P = [P_1, \dots, P_n]$ be the initial vector of plaintext group elements (derived from the base64-encoded flag).
*   $C = [C_1, \dots, C_n]$ be the final vector of ciphertext group elements.
*   $B \in \mathbb{F}_2^{n \times n}$ be the binary matrix `mat` generated by `gen_random_mat`.

The `hash_msg` function applies this transformation 10 times. So, if $P$ is the initial plaintext vector and $C$ is the final ciphertext vector, the overall operation can be written as:

$$
C = \left( B^{10} \cdot P \right)_G
$$

where the "multiplication" $B^{10} \cdot P$ is interpreted as a sequence of group product operations as defined above, governed by the matrix $B^{10}$. Our goal is to find $P$ given $C$ and $B$.

### 2. Linearization via Quotient Group

The group $G$ is non-abelian, this makes direct reversal difficult. However, we can exploit its structure by considering the quotient group $G/Z(G)$, where $Z(G)$ is the center of $G$.

This was mentioned above at the "Analysis of the Challenge Script" section, $Z(G) \cong C_2 \times C_2$ (or $\mathbb{F}_2^2$) and the quotient group: $G/Z(G) \cong C_2 \times C_2 \times C_2 \times C_2$ (or $\mathbb{F}_2^4$). and crucially, $G/Z(G)$ is abelian.

We define a natural homomorphism:
$$
\phi: G \to G/Z(G)
$$
By identifying $G/Z(G)$ with the vector space $\mathbb{F}_2^4$, $\phi(g)$ can be seen as a 4-bit vector.
Since $\phi$ is a homomorphism and its codomain $G/Z(G)$ is abelian (where the group operation becomes vector addition over $\mathbb{F}_2$, i.e., XOR), the transformation rule becomes linear:

$$
\phi(V_{\text{out}}[i]) = \phi\left(\prod_{j \mid B_{ij} = 1} V_{\text{in}}[j]\right) = \sum_{j \mid B_{ij} = 1} \phi(V_{\text{in}}[j])
$$
where the sum $\sum$ denotes component-wise XOR (addition in $\mathbb{F}_2^4$).

Let:
*   $\mathbf{P'}$ be the vector in $(\mathbb{F}_2^4)^n \cong \mathbb{F}_2^{4n}$ formed by concatenating all $\phi(P_k)$ for $k=\{1,\dots, n \}$.
*   $\mathbf{C'}$ be the vector in $(\mathbb{F}_2^4)^n \cong \mathbb{F}_2^{4n}$ formed by concatenating all $\phi(C_k)$ for $k=\{1,\dots, n\}$.
*   $B_{\text{eff}} = B^{10}$ be the effective matrix after 10 rounds.
*   $I_4$ be the $4 \times 4$ identity matrix over $\mathbb{F}_2$.

The relationship between the (mapped) plaintext and ciphertext after 10 rounds becomes a linear system over $\mathbb{F}_2$:

$$
\mathbf{C'} = \left( B_{\text{eff}} \otimes I_4 \right) \cdot \mathbf{P'}
$$

where $\otimes$ is the Kronecker product. This system can be solved for $\mathbf{P'}$ using techniques like Gaussian elimination. Solving this gives us $\phi(P_k)$ for each $k$. This means we know the coset $P_k Z(G)$ for each plaintext element $P_k$. Let $R_k$ be a chosen representative from this coset, so $P_k$ can be written as $P_k = R_k \cdot z_k$ for some unknown $z_k \in Z(G)$.

### 3. Solving for Central Elements

From the previous step, we have determined coset representatives $R_k$ such that $P_k = R_k \cdot z_k$, where each $z_k$ is an unknown element from the center $Z(G)$.
Substitute this form of $P_j$ back into the original group transformation (using $B_{\text{eff}}$):

$$
C_i = \prod_{j \mid B_{\text{eff}, ij} = 1} (R_j z_j)
$$

Since elements $z_j \in Z(G)$ commute with all elements in $G$ (including all $R_l$ and other $z_l$), we can rearrange the terms:

$$
C_i = \left( \prod_{j \mid B_{\text{eff}, ij} = 1} R_j \right) \cdot \left( \prod_{j \mid B_{\text{eff}, ij} = 1} z_j \right)
$$

Let $C'_{i, \text{known}} = \prod_{j \mid B_{\text{eff}, ij} = 1} R_j$. This part consists of known values (the $R_j$ are known from solving the first linear system). We can then isolate the product of the unknown central elements:

$$
Z''_i := (C'_{i, \text{known}})^{-1} \cdot C_i = \left( \prod_{j \mid B_{\text{eff}, ij} = 1} R_j \right)^{-1} \cdot C_i = \prod_{j \mid B_{\text{eff}, ij} = 1} z_j
$$

For this to be consistent, each $Z''_i$ must be an element of $Z(G)$.
Now we have a new system where all terms are in $Z(G)$. Since $Z(G)$ is isomorphic to $\mathbb{F}_2^2$, we can define another homomorphism:
$$
\psi: Z(G) \to \mathbb{F}_2^2
$$
This maps elements of the center to 2-bit vectors, and the group operation in $Z(G)$ (which is commutative) to vector addition (XOR) in $\mathbb{F}_2^2$. Applying $\psi$ to the equation for $Z''_i$:

$$
\psi(Z''_i) = \psi\left(\prod_{j \mid B_{\text{eff}, ij} = 1} z_j\right) = \sum_{j \mid B_{\text{eff}, ij} = 1} \psi(z_j)
$$

This is another system of linear equations over $\mathbb{F}_2$. Let $\mathbf{z'}$ be the vector in $(\mathbb{F}_2^2)^n \cong \mathbb{F}_2^{2n}$ formed by concatenating all $\psi(z_j)$, and $\mathbf{Z''}$ be the vector formed by concatenating all $\psi(Z''_i)$. The system is:
$$
\mathbf{Z''} = (B_{\text{eff}} \otimes I_2) \cdot \mathbf{z'}
$$
(Or, more simply, if we consider each $\psi(z_j)$ as a pair of equations, the matrix for each component is $B_{\text{eff}}$.)
Solving this system yields $\psi(z_j)$, which uniquely determines each $z_j \in Z(G)$.

If either of the linear systems ($P_k Z(G)$ or $z_k$) is under-determined (i.e., has free variables), we might have multiple candidate solutions. The known flag prefix `grey{` can be crucial here. The first few characters of the flag, after base64 encoding, provide the true values for the initial $P_k$. These known $P_k$ also mean their $\phi(P_k)$ and $z_k$ components are known, which can help resolve ambiguities in the linear systems or verify solutions. In this challenge, the script handles potential ambiguities by exploring choices for free variables.

### 4. Reconstruct Plaintext

Once both components are determined for each $k$:
*   The coset representative $R_k$ (derived from solving for $\phi(P_k)$)
*   The central element $z_k \in Z(G)$ (derived from solving for $\psi(z_k)$)

We can reconstruct the original plaintext group elements:
$$
P_k = R_k \cdot z_k
$$

Finally, the vector of group elements $P = [P_1, \dots, P_n]$ is converted back from group elements to their corresponding indices, then to a base64 string. This base64 string is padded with `=` if necessary, then decoded to bytes. This candidate flag is validated by re-applying the `hash_msg` function and comparing its output with the given ciphertext `ct`.

### Summarization
Our approach breaks the problem into two linear systems:

1.  **Linear system in the quotient group $G/Z(G)$** (to find coset representatives):
    Let $\mathbf{P'}$ be the concatenation of $\phi(P_k)$ and $\mathbf{C'}$ be the concatenation of $\phi(C_k)$.
    $$
    \mathbf{C'} = \left( B^{10} \otimes I_4 \right) \cdot \mathbf{P'} \quad (\text{over } \mathbb{F}_2)
    $$
    Solving this gives $R_k$ (representatives for $P_k Z(G)$).

2.  **Linear system in the center $Z(G)$ (to find central elements $z_k$):**

    Define: $$Z''_i := \left( \prod_{j \mid (B^{10})_{ij} = 1} R_j \right)^{-1} \cdot C_i$$
    Let $\mathbf{z'}$ be the concatenation of $\psi(z_j)$ and $\mathbf{Z''}$ be the concatenation of $\psi(Z''_i)$.
    $$
    \mathbf{Z''} = \left( B^{10} \otimes I_2 \right) \cdot \mathbf{z'} \quad (\text{over } \mathbb{F}_2)
    $$
    (Alternatively, $\psi(Z''_i) = \sum_{j} (B^{10})_{ij} \cdot \psi(z_j)$ for each component of the $\mathbb{F}_2^2$ vectors.)
    Solving this gives each $z_j$.

By doing this, it sidesteps the complexities of the non-abelian group $G$ by working within its more structured abelian components.

## Implementation

Well, I knew how i can approach the problem mathematically, but I have encountered the "Skill Issue" moment in my Sage knowledge.

I finally realized what i did wrong on my solver script after reading through Soon-hari's solver script. Still messed up the tqdm lol 

```sage=
"""
Solver blah blah for group theory
"""

import random
import base64
from functools import reduce
from tqdm import tqdm

# Constants
SEED = int(0xcaffe * 3 * 0xc0ffee)
out = "aO3qDbHFoittWTN6MoUYw9CZiC9jdfftFGw1ipES89ugOwk2xCUzDpPdpBWtBP3yarjNOPLXjrMODD"
b64encode_map = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0987654321+/"
TARGET_PREFIX = b"grey"

def initialize_group():
    """Initialize the permutation group and precompute elements"""
    generators = [
        [(1,2,3,4),(5,6,7,8),(9,10,11,12),(13,14,15,16),(17,18,19,20),(21,22,23,24),(25,26,27,28),(29,30,31,32)],
        [(1,24,3,22),(2,23,4,21),(5,14,7,16),(6,13,8,15),(9,27,11,25),(10,26,12,28),(17,31,19,29),(18,30,20,32)],
        [(1,5,12,30),(2,6,9,31),(3,7,10,32),(4,8,11,29),(13,25,19,21),(14,26,20,22),(15,27,17,23),(16,28,18,24)],
        [(1,26),(2,27),(3,28),(4,25),(5,14),(6,15),(7,16),(8,13),(9,23),(10,24),(11,21),(12,22),(17,31),(18,32),(19,29),(20,30)]
    ]
    
    group = PermutationGroup(generators)
    elements = list(group)
    element_to_index = {element: idx for idx, element in enumerate(elements)}
    squares = set(g^2 for g in group)
    identity = group.identity()
    
    return group, elements, element_to_index, squares, identity

def create_decode_map(encode_map):
    """Create decode mapping for custom base64"""
    return {char: idx for idx, char in enumerate(encode_map)}

def gen_matrix(seed, size):
    """Generate random binary matrix with given seed"""
    random.seed(seed)
    random_data = random.choices([0, 1], k=size*size)
    return matrix(GF(2), size, size, random_data)

def vec_mul(matrix, vector):
    """Multiply matrix by vector over the permutation group"""
    return [reduce(lambda x, y: x * y, (elem^bit for elem, bit in zip(vector, row))) 
            for row in matrix]

def gaussian_eliminiation(matrix_gf2, vector_perm):
    size = matrix_gf2.nrows()
    # Create a copy using matrix construct
    mat_copy = Matrix(GF(2), matrix_gf2)
    vec_copy = vector_perm[:]
    
    for i in range(size - 1):
        # Find pivot row
        pivot_row = None
        for j in range(i, size):
            if mat_copy[j, i] == 1:
                pivot_row = j
                break
        
        if pivot_row is None:
            print(f"Gaussian elimination failed you at column {i}")
            exit()
            
        # Swap rows 
        if pivot_row != i:
            mat_copy.swap_rows(i, pivot_row)
            vec_copy[i], vec_copy[pivot_row] = vec_copy[pivot_row], vec_copy[i]
        
        # Get pivot element and normalize
        pivot_element = mat_copy[i, i]
        mat_copy.rescale_row(i, pivot_element)
        vec_copy[i] = vec_copy[i]^pivot_element
        
        # KILL DA other rows
        for j in range(size):
            if j != i:
                multiplier = mat_copy[j, i]
                if multiplier != 0:
                    mat_copy.add_multiple_of_row(j, i, -multiplier)
                    vec_copy[j] = vec_copy[j] * vec_copy[i]^(4 - int(multiplier))
    
    return mat_copy, vec_copy

def solve():
    print("Initializing group and mappings...")
    group, elements, element_to_index, squares, identity = initialize_group()
    decode_map = create_decode_map(b64encode_map)
    
    print("Decoding ciphertext...")
    ciphertext_elements = [elements[decode_map[char]] for char in out]
    vector_length = len(ciphertext_elements)
    
    print("Generating rando matrix...")
    base_matrix = gen_matrix(SEED, vector_length)
    
    print("Computing matrix...")
    matrix_power_10 = base_matrix^10
    
    print("Performing Gaussian elimination...")
    reduced_matrix, reduced_vector = gaussian_eliminiation(
        matrix_power_10, ciphertext_elements[:]
    )
    
    print("Testing da candidates...")
    base_matrix_list = [[int(base_matrix[i, j]) for j in range(vector_length)] 
                        for i in range(vector_length)]
    
    for candidate_idx in tqdm(range(17), desc="Testing da candidates"):
        # Generate candidate by setting free variable
        candidate = reduced_vector[:]
        free_variable = elements[candidate_idx]
        
        for j in range(vector_length):
            if reduced_matrix[j, vector_length - 1] or j == vector_length - 1:
                candidate[j] *= free_variable
        
        # Matrix Mutiply
        current_vector = candidate[:]
        for _ in range(10):
            current_vector = vec_mul(base_matrix_list, current_vector)
        
        # Check if differences are all squares
        differences = [ciphertext_elements[i] / current_vector[i] 
                      for i in range(vector_length)]
        
        if not all(diff^2 == identity for diff in differences):
            continue
        
        # Solve for differences
        diff_matrix, diff_vector = gaussian_eliminiation(
            matrix_power_10, differences[:]
        )
        
        if diff_vector[-1] != identity:
            continue
        
        # Generate using squares
        for square_perm in squares:
            final_candidate = diff_vector[:]
            
            for j in range(vector_length):
                if diff_matrix[j, vector_length - 1] or j == vector_length - 1:
                    final_candidate[j] *= square_perm
            
            # Combine with original candidate
            answer_candidate = [final_candidate[i] * candidate[i] 
                              for i in range(vector_length)]
            
            # Verification 
            verification_vector = answer_candidate[:]
            for _ in range(10):
                verification_vector = vec_mul(
                    base_matrix_list, verification_vector
                )
            
            if verification_vector != ciphertext_elements:
                continue
            
            # Decode it
            try:
                encoded_answer = ''.join(b64encode_map[element_to_index[elem]] 
                                       for elem in answer_candidate)
                decoded_bytes = base64.b64decode(encoded_answer + "==")
                
                if decoded_bytes.startswith(TARGET_PREFIX):
                    return decoded_bytes
                    
            except (KeyError, ValueError):
                continue
    
    return None

def main():
    try:
        result = solve()
        if result:
            print(f"Solution found: {result}")
        else:
            print("Bah~ No flagz for ya mate")
    except Exception as e:
        print(f"Error: {e}")
        raise
```

## Side Note 

I hope you enjoyed my write up :) First time publishing it and shared a link in public, I'm quite afraid that it might be not helpful or it's misleading. If you have any comments, please send me a dm on Discord @Mardcelo.

When I was working on this challenge, I was reviewing group theory again, which I was able to learn more and enlighten myself.

This oppertunity also gave me to solve few more problems in Putnam exam that had problems related to group theory in the aspects of chemistry, and also binary operations, its really fun problems. 

Thank you for reading and have a amazing morning/evening/afternoon/night!













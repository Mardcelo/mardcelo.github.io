# [GreyCat CTF] Not a Permutation Matrix 

- [name=Mard (Rick) Kim]
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

## Background

I was playing a CTF with RubiyaLab, and with CryptoPros [@1s0m0rph1sm](https://x.com/1s0m0rph1sm), and [@playteddypicker](https://x.com/499_addiction) (Full kudos to them, really smart guys) 

I was  sick during the time, so I didn't really had energy to solve and contribute that much. I was only able to up-solve, after the CTF was over. The reason why I'm blog posting is because this challenge was quite interesting where it used specific group.

## Introduction to the Permutation Group

This challenge used a specific group denoted with 
$$Q_8 \rtimes_5 D_4$$ 

which is a p-group, metabelian, nilpotent (class 2), monomial. 

You would say huh?? and so do I. 

The basic meaning of this is that $Q_8 \rtimes_5 D_4$ means a semidirect product ($\rtimes$) of two groups: $Q_8$ (the quaternion group of order $8$) and $D_4$ (the Dhihedral group of order $8$). The $5$ subscript indicates a particular way these groups are combined. Which it signifies a specific action of $D_4$ on $Q_8$ that determines how the semidirect product is formed. It's the 1st semidriect product out of several possible ways to combine $Q_8$ and $D_4$. The full database can be found in this [link](https://people.maths.bris.ac.uk/~matyd/GroupNames/1/C4sC4.html)
 

## Analysis of the Code 

Going back to our challenge, We were given a sage script that we had a function called `hash_msg` which took the input, base64 encoded it, converted characters to elements of a specific permutation group `G`, and then iteratively applied a matrix-vector-like multiplication using group operations. The final group elements were then converted back to a base64 string. 

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

This might be true, which non-abelian operations can be difficult to reverse. But there is a plot twist. 

While `G` itself is non-ablian, we can consider its structure "modulo its center." The center of a group, $Z(G)$, consists of all elements that commute with every element in $G$. To understand the trick, we need to look at the quotient group $Q = G \backslash Z(G)$.

For the specific group `G` used in this challenge $Q_8 \rtimes_5 D_4$ is isomorphic to $C_2 \times C_2$ (the Klein four-group, or $F_2^2$ as a vector space). More importantly, the quotient group $Q=G \backslash Z(G)$ turns out to be isomorphic to $(C_2)^4$ (the elementary abelian group of order 16, or $F_2^4$ as a 4-dimensional vector space over the field of two elements, $F^2$). This is good news for us because $F_2^4$ is abelian, and its operations are simple vector additions (XORs).

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


## Making a Solver Script

Mentioned at beginning of the blog post, the challenge asks us to reverse the `hash_msg` function.

If we let $P$ be the vector of initial group elements (from the base64-encoded flag) and $C$ be the vector of final ciphertext group elements, the main operation repeated 10 times is: 

$$V_{out}[i] = \prod_{j \text{ where } B_{ij}=1} V_{in}[j]$$

where $B$ is the binary matrix `mat`. So we are effectively trying to solve $C = (B^{10}) \cdot P$ in the context of this group operation.

As discussed in the group theory background, is that while $G$ itself is non-abelian, its quotient group $G/Z(G)$ (where $Z(G)$ is the center of $G$) is abelian, and isomorphic to $(\mathbb{F}_2)^4$. This structure allows us to linearize the problem. 

First, we map to the quotient group and linearization. We utilize the natural homomorphism $\phi: G \rightarrow G/Z(G) \cong (\mathbb{F}_2)^4$. This map takes an element $g \in G$ to its coset $gZ(G)$.

Which we then identify with a 4-bit vector over $\mathbb{F}_2$. Since $\phi$ is a homomorphism into an abelian group (where the operation can be seen as XOR, or addition in $\mathbb{F}_2$), the original product operation transforms beautifully:

$$\phi(V_{out}[i]) = \phi(\prod_{j | B_{ij}=1} V_{in}[j]) = \sum_{j | B_{ij}=1} \phi(V_{in}[j])$$ 

(where the sum is component-wise XOR)

Let $P'_k = \phi(P_k)$ and $C'_i = \phi(C_i)$ be the 4-bit vectors corresponding to the plaintext and ciphertext elements, respectively. 

The transformation for one iteration becomes $C'_i = \sum_{j | B_{ij}=1} P'_j$. 

If we concatenate all $P'_k$ into a long vector $\mathbf{P'}$ of length $4 \cdot sz$ (where $sz$ is the length of the message), and similarly for $\mathbf{C'}$, this transformation can be represented by a large $(4 \cdot sz) \times (4 \cdot sz)$ matrix $M_L = B \otimes I_4$, where $I_4$ is the $4 \times 4$ identity matrix and $\otimes$ is the Kronecker product.

The hash function applies this 10 times. Thus, the overall system relating the initial plaintext vectors (modulo $Z(G)$) to the final ciphertext vectors (modulo $Z(G)$) is:

$$\mathbf{C'} = (B \otimes I_4)^{10} \mathbf{P'} = (B^{10} \otimes I_4) \mathbf{P'}$$

Let $B_{eff} = B^{10}$. The system is $\mathbf{C'} = (B_{eff} \otimes I_4) \mathbf{P'}$. This is a standard system of linear equations over $\mathbb{F}_2$. 

We can solve this for $\mathbf{P'}$ using Gaussian elimination. This determines each $P_k$ up to an element of $Z(G)$; specifically, it tells us the coset $P_k Z(G)$ for each $k$.

Once we have solved for $\phi(P_k)$ for all $k$, we can start to determinine the central components.

We know a coset representative $R_k$ such that $P_k = R_k z_k$, where $z_k$ is an unknown element from the center $Z(G)$. We substitute this back into the original group equation using the effective matrix $B_{eff} = B^{10}$:

$$C_i = \prod_{j | (B_{eff})_{ij}=1} (R_j z_j)$$

Since elements $z_j \in Z(G)$ commute with all elements in $G$, we can rearrange this:

$$C_i = \left(\prod_{j | (B_{eff})_{ij}=1} R_j\right) \cdot \left(\prod_{j | (B_{eff})_{ij}=1} z_j\right)$$

Let 
$$C'_{i,group} = \prod_{j | (B_{eff})_{ij}=1} R_j$$

Then we have 

$$C_i = C'_{i,group} \cdot \prod_{j | (B_{eff})_{ij}=1} z_j$$


This implies 

$$(C'_{i,group})^{-1} C_i = \prod_{j | (B_{eff})_{ij}=1} z_j$$

Let $Z''_i = (C'_{i,group})^{-1} C_i$ For our $R_j$ to be consistent, each $Z''_i$ must be an element of $Z(G)$.

Now we have a new system involving only elements from the center: 

$$Z''_i = \prod_{j | (B_{eff})_{ij}=1} z_j$$

Since $Z(G) \cong C_2 \times C_2$, we can use another homomorphism $\psi: Z(G) \rightarrow (\mathbb{F}_2)^2$, which maps elements of the center to 2-bit vectors and the group operation in $Z(G)$ to XOR. Applying $\psi$:

$$\psi(Z''_i) = \sum_{j | (B_{eff})_{ij}=1} \psi(z_j)$$

This is another system of linear equations over $\mathbb{F}_2$, this time for the 2-bit vectors $\psi(z_j)$. The matrix governing this system is again $B_{eff}$. 

We solve this system for $\psi(z_j)$, which uniquely determines each $z_j \in Z(G)$.

**3. Reconstruction and Validation**

With both the coset representatives $R_k$ (from $\phi(P_k)$) and the central elements $z_k$ determined, we can reconstruct the original plaintext elements $P_k = R_k z_k$.

If either of the linear systems was under-determined (i.e., had free variables), we would have some choices to make. 

The known flag prefix `grey{` is important here. 

The first few characters of the flag, after base64 encoding, give us the first few $P_k$. 

This means we know their $\phi(P_k)$ and $z_k$ components, which can be used to resolve ambiguities in the linear systems. In this challenge, the systems turned out to be uniquely solvable or easily resolvable with the prefix.

Finally, the vector of group elements $P$ is converted back to a base64 string, padded if necessary, and decoded to bytes. 

This candidate flag is then validated by re-hashing it using the provided `hash_msg` function and comparing the output to the given ciphertext `ct`. If they match, we have successfully recovered the flag.

This method, by breaking down the problem into two manageable linear systems over $\mathbb{F}_2$, interestingly sidesteps the complexities of the non-abelian group $G$ by working within its more structured abelian components (the quotient $G/Z(G)$ and the center $Z(G)$ itself).

Well after this thinking in my mind, I had encountered the "Skill Issue" in my sage knowledge. I finally realized what i did wrong on my solver script which I have been suffering from Sage after reading through Soon-hari's script 

```sage
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
    """Initialize the permutation group and precompute elements."""
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
    """Create decode mapping for custom base64."""
    return {char: idx for idx, char in enumerate(encode_map)}

def gen_matrix(seed, size):
    """Generate random binary matrix with given seed."""
    random.seed(seed)
    random_data = random.choices([0, 1], k=size*size)
    return matrix(GF(2), size, size, random_data)

def vec_mul(matrix, vector):
    """Multiply matrix by vector over the permutation group."""
    return [reduce(lambda x, y: x * y, (elem^bit for elem, bit in zip(vector, row))) 
            for row in matrix]

def gaussian_eliminiation(matrix_gf2, vector_perm):
    """
    Gaussian elimination over GF(2) with permutation group vector.
    
    Args:
        matrix_gf2: Matrix over GF(2)
        vector_perm: Vector of permutation group elements
        
    Returns:
        Tuple of (reduced_matrix, reduced_vector)
    """
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
                    # For permutation group: vec[j] = vec[j] * vec[i]^(4 - multiplier)
                    vec_copy[j] = vec_copy[j] * vec_copy[i]^(4 - int(multiplier))
    
    return mat_copy, vec_copy

def solve():
    """Main solving function"""
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
    # Pre compute matrix multiplication for efficiency
    base_matrix_list = [[int(base_matrix[i, j]) for j in range(vector_length)] 
                        for i in range(vector_length)]
    
    for candidate_idx in tqdm(range(17), desc="Testing da candidates"):
        # Generate candidate by setting free variable
        candidate = reduced_vector[:]
        free_variable = elements[candidate_idx]
        
        for j in range(vector_length):
            if reduced_matrix[j, vector_length - 1] or j == vector_length - 1:
                candidate[j] *= free_variable
        
        # Matrix multiplication 
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
        
        # Generate final candidates using squares
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
            
            # Convert to base64 and decode
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
    """Main entry"""
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















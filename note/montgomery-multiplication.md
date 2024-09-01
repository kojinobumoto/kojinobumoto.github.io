# reference
- [Montgomery Multiplication (Algorithmica / HPC)](https://en.algorithmica.org/hpc/number-theory/montgomery/)
- [Montgomery Multiplication (cp-algorithms.com)](https://cp-algorithms.com/algebra/montgomery_multiplication.html)
- [Montgomery Multiplication Explained (Fast Modular Multiplication)](https://codeforces.com/blog/entry/103374)
- [Montgomery reduction (DSZQUP XJLJ)](https://cryptography.fandom.com/wiki/Montgomery_reduction)
- [Open Quantum Safe (github)](https://github.com/open-quantum-safe)

# Before getting started
I'll use the same notation in [this page](https://en.algorithmica.org/hpc/number-theory/montgomery/) to represent the number $x$ and multiplication $*$ in the Montgomery space, and $\text{`}\cdot\text{`}$ as the "normal" multiplication.

i.e)
- The representative of a number $x$ in the Montgomery space is

$$
\overline{x} = x \cdot r \mod{n}
$$

- The Montgomery Multiplication is

$$
\overline{x} * \overline{y} = \overline{x} \cdot \overline{y} \cdot r^{-1} \mod{n}
$$

- $n$ and $r$ are integers such that $r > n$ and $r$ is coprime to $n$.

# The main part of this note.
I was curious about if the Montgomery Multiplication was 

$$
\overline{x} * \overline{y} = \overline{x} \cdot \overline{y} \cdot r^{-1} \mod{p}
$$

then, 

how the actual implementation becomes as follows (for example, *modp_montymul()* in [sig/falcon/pqclean_falcon-1024_aarch64/keygen.c](https://github.com/open-quantum-safe/liboqs/blob/main/src/sig/falcon/pqclean_falcon-1024_aarch64/keygen.c#L716-L726) ).
```
/*
 * Montgomery multiplication modulo p. The 'p0i' value is -1/p mod 2^31.
 * It is required that p is an odd integer.
 */
static inline uint32_t
modp_montymul(uint32_t a, uint32_t b, uint32_t p, uint32_t p0i) {
    uint64_t z, w;
    uint32_t d;

    z = (uint64_t)a * (uint64_t)b;
    w = ((z * p0i) & (uint64_t)0x7FFFFFFF) * p;
    d = (uint32_t)((z + w) >> 31) - p;
    d += p & -(d >> 31);
    return d;
}
```

Now I think I understood "why", so, I'll keep my note in this page.

## Explanation
We want to compute 

$$
\overline{x} \cdot \overline{y} \cdot r^{-1} \mod{p}
$$ 

where $r = 2^{31}$ and $p$ is a prime such that $2^{30} < p < 2^{31}$ .

Now, for some integer $m$

$$
\overline{x} \cdot \overline{y} \cdot r^{-1} \mod{p} = (\overline{x} \cdot \overline{y} + m \cdot p) \cdot r^{-1} \mod{p}
$$

### Let's see why.
Let 

$$
C = \overline{x} \cdot \overline{y} + m \cdot p
$$

then,

$$
C \cdot r^{-1} = (\overline{x} \cdot \overline{y} + m \cdot p) \cdot r^{-1} = \overline{x} \cdot \overline{y} \cdot r^{-1} + m \cdot p \cdot r^{-1}
$$

$$
\begin{aligned}
C \cdot r^{-1} \mod{p} \\
\qquad &= (\overline{x} \cdot \overline{y} \cdot r^{-1} + m \cdot p \cdot r^{-1}) \mod{p} \\
\qquad &= ( (\overline{x} \cdot \overline{y} \cdot r^{-1} \mod{p}) + (m \cdot p \cdot r^{-1} \mod{p}) ) \mod{p} \\
\qquad &= (\overline{x} \cdot \overline{y} \cdot r^{-1} \mod{p})  \mod{p}   \quad (\text{ * since}  \quad m \cdot p \cdot r^{-1} \text{ is divisible by } p \text{,} \quad m \cdot p \cdot r^{-1} \mod{p} = 0) \\
\qquad &= \overline{x} \cdot \overline{y} \cdot r^{-1} \mod{p}  \quad  (\text{ * since} \quad (a \mod{b}) \mod{b} = a \mod{b})
\end{aligned}
$$

### We want $m$ such that,

$$
(\overline{x} \cdot \overline{y} + m \cdot p) \mod{r} = 0
$$

so,

$$
\overline{x} \cdot \overline{y} + m \cdot p \equiv 0 \mod{r} 
$$

that is equivalent

$$
\overline{x} \cdot \overline{y} + m \cdot p = k \cdot r 
$$

for some integer $k$.

We can solve for $m$:

$$
m \cdot p = k \cdot r  - \overline{x} \cdot \overline{y} 
$$

$$
m \cdot p \equiv -\overline{x} \cdot \overline{y} \mod{r} 
$$

Now, since $p \cdot p^{-1} \equiv 1 \mod{r}$  (* $p$ and $r$ are coprime), by multiplying both sides of above equation by $p^{-1}$, we get

$$
m \equiv -\overline{x} \cdot \overline{y} \cdot p^{-1} \mod{r}
$$

therefore,

$$
m = (-\overline{x} \cdot \overline{y} \cdot p^{-1}) \mod{r}
$$

### As a result of above discussion, we get,

$$
\begin{aligned}
\overline{x} \cdot \overline{y} \cdot r^{-1} \mod{p} \\
\qquad &= (\overline{x} \cdot \overline{y} + m \cdot p) \cdot r^{-1} \mod{p} \\
\qquad &= ( (\overline{x} \cdot \overline{y} + ((-\overline{x} \cdot \overline{y} \cdot p^{-1}) \mod{r}) \cdot p) \cdot r^{-1} ) \mod{p}
\end{aligned}
$$

### Now, let's compare with the actual implementation.
`z = (uint64_t)a * (uint64_t)b;` is equivalent to $\overline{x} \cdot \overline{y}$.

`(z * p0i)` is equivalent to $(-\overline{x} \cdot \overline{y} \cdot p^{-1})$, because the 'p0i' value is $-1/p \mod{r}$ (modular inverse of $p$ (i.e. $p^{-1}$)).

Now, `0x7FFFFFFF` is the 31 bit sequence of **1** (**1111....**).

So, `& (uint64_t)0x7FFFFFFF` is equivalent to **"mod r"**. Since $r=2^{31}$ (pow 2 31), taking the lower 31 bit of $(-\overline{x} \cdot \overline{y} \cdot p^{-1})$ with `0x7FFFFFFF` means getting the remainings divided by $2^{31}$.

Therefore, `(z * p0i) & (uint64_t)0x7FFFFFFF)` is equivalent to $((-\overline{x} \cdot \overline{y} \cdot p^{-1}) \mod{r})$, 

and `w = ((z * p0i) & (uint64_t)0x7FFFFFFF) * p;` is equivalent to $((-\overline{x} \cdot \overline{y} \cdot p^{-1}) \mod{r}) \cdot p)$.

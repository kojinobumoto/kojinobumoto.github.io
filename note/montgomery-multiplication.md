# reference
- [Montgomery Multiplication (Algorithmica / HPC)](https://en.algorithmica.org/hpc/number-theory/montgomery/)
- [Montgomery Multiplication (cp-algorithms.com)](https://cp-algorithms.com/algebra/montgomery_multiplication.html)
- [Montgomery Multiplication Explained (Fast Modular Multiplication)](https://codeforces.com/blog/entry/103374)
- [Montgomery reduction (DSZQUP XJLJ)](https://cryptography.fandom.com/wiki/Montgomery_reduction)
- [Open Quantum Safe (github)](https://github.com/open-quantum-safe)

# Begore get started
I'll use the same notation in [this page](https://en.algorithmica.org/hpc/number-theory/montgomery/) to represente the number $x$ and multiplication $*$ in the Montgomery space, and $\cdot$ as the "normal" multiplication.

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

# The body of this note.
So, I was curious about if the Montgomery Multiplication was $\overline{x} * \overline{y} = \overline{x} \cdot \overline{y} \cdot r^{-1} \mod{n}$, then, how the actual implementation becomes as following (for example in [sig/falcon/pqclean_falcon-1024_aarch64/keygen.c](https://github.com/open-quantum-safe/liboqs/blob/main/src/sig/falcon/pqclean_falcon-1024_aarch64/keygen.c#L716-L726) )
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
So, I want to compute $\overline{x} \cdot \overline{y} \cdot r^{-1} \mod{p}$, where $r = 2^{-31}$ and $p$ is a prime such that $2^{30} < p < 2^{31}$ .
As explained in [this page](https://codeforces.com/blog/entry/103374), in general, if I can find some integer $m$ such that $(\overline{x} \cdot \overline{y}) \cdot r^{-1} ¥mod{p} = 0$ (


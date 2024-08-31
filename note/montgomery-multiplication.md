# reference
- [Montgomery Multiplication (Algorithmica / HPC)](https://en.algorithmica.org/hpc/number-theory/montgomery/)
- [Montgomery Multiplication (cp-algorithms.com)](https://cp-algorithms.com/algebra/montgomery_multiplication.html)
- [Montgomery Multiplication Explained (Fast Modular Multiplication)](https://codeforces.com/blog/entry/103374)
- [Montgomery reduction (DSZQUP XJLJ)](https://cryptography.fandom.com/wiki/Montgomery_reduction)
- [Open Quantum Safe (github)](https://github.com/open-quantum-safe)

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




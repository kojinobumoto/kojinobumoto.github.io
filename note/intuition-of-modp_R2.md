---
layout: post
title: intuition of modp_R2.
katex: True
---
# intuition of [modp_R2](https://github.com/open-quantum-safe/liboqs/blob/main/src/sig/falcon/pqclean_falcon-1024_aarch64/keygen.c#L728-L757)

```
/*
 * Compute R2 = 2^62 mod p.
 */
static uint32_t
modp_R2(uint32_t p, uint32_t p0i) {
    uint32_t z;

    /*
     * Compute z = 2^31 mod p (this is the value 1 in Montgomery
     * representation), then double it with an addition.
     */
    z = modp_R(p);
    z = modp_add(z, z, p);

    /*
     * Square it five times to obtain 2^32 in Montgomery representation
     * (i.e. 2^63 mod p).
     */
    z = modp_montymul(z, z, p, p0i);
    z = modp_montymul(z, z, p, p0i);
    z = modp_montymul(z, z, p, p0i);
    z = modp_montymul(z, z, p, p0i);
    z = modp_montymul(z, z, p, p0i);

    /*
     * Halve the value mod p to get 2^62.
     */
    z = (z + (p & -(z & 1))) >> 1;
    return z;
}
```

`modp_R(p)` computes $$\text{"}2^{31} \mod{p}\text{"}$$, which is `1` in Montgomery representation (i.e. $$z = 1 \cdot r \mod{p}$$, where $$r=2^{31}$$ and $$2^{30} < p < 2^{31}$$).

`modp_add(z, z, p)` doubles $$z$$,
i.e. 

$$
\begin{aligned}
z = 2 \cdot 2^{31} \mod{p} \\
&=2^{32} \mod{p}.
\end{aligned}
$$

Since the Montgomery Multiplication is

$$
\begin{aligned}
\overline{x} * \overline{y} = \overline{x} \cdot \overline{y} \cdot r^{-1} \mod{p} \\
\\
\text{where } n \text{ and } r \text{ are integers such that } r > p \text{ and } r \text{ is coprime to } p.
\end{aligned}
$$

, the 1st `modp_montymul(z, z, p, p0i)` computes, 

$$
\begin{aligned}
z * z = z \cdot z \cdot r^{-1} \mod{p} \\
&= ((2^{32} \mod{p}) \cdot (2^{32} \mod{p}) \cdot (2^{-31})) \mod{p} \\
&= \[ \lbrace(2^{32} \mod{p}) \cdot (2^{32} \mod{p}) \mod{p} \rbrace \cdot (2^{-31} \mod{p} ) \] \mod{p} \\
&= ( (2^{64} \mod{p}) \cdot (2^{-31} \mod{p} )) \mod{p}  \quad \text{ (*1) } \\
&= 2^{64} \cdot 2^{-31} \mod{p} \\
&= 2^{33} \mod{p} \\
\\
\text{ (*1)  } \quad 
(a \mod{b}) \mod{b} 
&= a \mod{b}
\end{aligned}
$$

The 2nd `modp_montymul(z, z, p, p0i)` will be

$$
\begin{aligned}
z * z = z \cdot z \cdot r^{-1} \mod{p} \\
&= ((2^{33} \mod{p}) \cdot (2^{33} \mod{p}) \cdot (2^{-31})) \mod{p} \\
&= \[ \lbrace(2^{33} \mod{p}) \cdot (2^{33} \mod{p}) \mod{p} \rbrace \cdot (2^{-31} \mod{p} ) \] \mod{p} \\
&= ( (2^{66} \mod{p}) \cdot (2^{-31} \mod{p} )) \mod{p} \\
&= 2^{66} \cdot 2^{-31} \mod{p} \\
&= 2^{35} \mod{p} \\
\end{aligned}
$$

Likewise, the 3rd, 4th and 5th `modp_montymul(z, z, p, p0i)` will be 

$$( (2^{70} \mod{p}) \cdot (2^{-31} \mod{p} )) \mod{p} \quad \rightarrow \quad 2^{39} \mod{p}$$

$$( (2^{78} \mod{p}) \cdot (2^{-31} \mod{p} )) \mod{p} \quad \rightarrow \quad 2^{47} \mod{p}$$

$$( (2^{94} \mod{p}) \cdot (2^{-31} \mod{p} )) \mod{p} \quad \rightarrow \quad 2^{63} \mod{p}$$

Finally we get $$2^{62}$$ by halving (`>> 1`)the value mod p.

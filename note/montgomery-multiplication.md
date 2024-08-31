So, the [Montgomery Multiplication](https://en.algorithmica.org/hpc/number-theory/montgomery/) in the Montgomery space is defined as

$$
\overline{x} * \overline{y} = \overline{x} \cdot \overline{y} \cdot r^{-1} \mod{n}
$$

where $\overline{x}$ is the representative of a number $x$ in the Montgomery space

$$
\overline{x} = x \cdot r \mod{n}
$$

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




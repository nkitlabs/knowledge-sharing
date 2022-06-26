# Muldiv on large integers

**Goal**: compute $\mathrm{muldiv}(x,y,z) = \lfloor \frac{x * y}{z} \rfloor \space ; \space x,y,z, \lfloor \frac{x * y}{z} \rfloor \lt 2^{256}$

**Steps**:

1. Compute the 512-bit product using mulmod

   $$
   \begin{aligned}
   xy = a*2^{256} + b
   \end{aligned}
   $$

2. Subtract the remainder of the $xy \space\mathrm{mod}\space z$ from the 512-bit product. So that $a*2^{256} + b - (xy \space \mathrm{mod}\space z)$ can be divided by $z$

   $$
   \begin{aligned}
   xy &= z*\mathrm{muldiv}(x,y,z) + xy \space\mathrm{mod}\space z \\
   \mathrm{muldiv}(x,y,z) &= (a*2^{256} + b - xy \space\mathrm{mod}\space z)/ z
   \end{aligned}
   $$

3. Remove powers of two from the fraction to make the denominator invertible $2^{256}$

   $$
   \begin{aligned}
   \mathrm{muldiv}(x,y,z) &= (a*2^{256} + b - xy \space\mathrm{mod}\space z)/ 2^i/ z'\space ; \space z' = z / 2^i \text{and } 2^i | z \\
   &= (a'*2^{256} + b') / z'
   \end{aligned}
   $$

4. multiply by the inverse of $z' \space\mathrm{mod}\space 2^{256}$.
   Noted: we must make sure that $\mathrm{muldiv}(x,y,z) \lt 2^{256}$

$$
\begin{aligned}
\mathrm{muldiv}(x,y,z) \space\mathrm{mod}\space 2^{256} &\equiv (a'*2^{256} + b')/z' \space\mathrm{mod}\space 2^{256} \\
 &\equiv (a'*2^{256} + b') * z'^{-1}\space\mathrm{mod}\space 2^{256} \\
 &\equiv b' * z'^{-1}\space\mathrm{mod}\space 2^{256}
\end{aligned}
$$

## Compute 512-bit product

The result of $x * y$ can be writen in the format of $a * 2^{256} + b$.

we can easily calculate $b$ by letting the result overflow.

To calculate $a$

$$
\begin{aligned}
xy \space\mathrm{mod}\space (2^{256}-1) &\equiv a*2^{256} + b\space\mathrm{mod}\space (2^{256}-1)\\
&\equiv a  \space\mathrm{mod}\space (2^{256}-1) + b \space\mathrm{mod}\space (2^{256}-1) \\
a \space\mathrm{mod}\space (2^{256}-1) &\equiv xy \space\mathrm{mod}\space (2^{256}-1) - b \space\mathrm{mod}\space (2^{256}-1)
\end{aligned}
$$

We can use that value $a$ because $a \lt 2^{256}-1$.

We can calculate $xy \space\mathrm{mod}\space (2^{256}-1)$
by using $\mathrm{mulmod}(x,y,2^{256}-1)$. See [here](#Mulmod-on-large-integers) how does it work.

## Compute inverse modulo $2^{256}$

**Goal**: Find $z'^{-1} \space\mathrm{mod}\space 2^{256}$

Assumption: $2 \space \nmid \space z'$

Simple solution:

From Inverse Modulo (see [here](#Inverse-Modulo)), $z'^{-1} \space\mathrm{mod}\space 2^{256} \equiv z^{2^{254}-1}\space\mathrm{mod}\space 2^{256}$

which use exponential multiplication in $O(lg\space n)$

Another Solution:

Observation: we can confirm that there is an inverse modulo of $z$ over modulo $2^i$

From Hensel's lifting lemma [here](https://en.wikipedia.org/wiki/Hensel%27s_lemma#Using_derivatives_for_lifting_roots),

let

$$
f(r_i) = z' * r_i - 1 \equiv 0 \space\mathrm{mod}\space 2^{2^i}
$$

We notice that,

$$
f(r_i) \equiv 0 \space\mathrm{mod}\space 2^{2^i} \text{ and } f'(r_i) \equiv z' \space\mathrm{mod}\space 2^{2^i} \not\equiv 0 \space\mathrm{mod}\space 2^{2^i}
$$

So, there is an integer $r_{i+1} \equiv r_i - f(r_i) * (f'^{-1}(r_i) \space\mathrm{mod}\space 2^{2^i}) \space\mathrm{mod}\space 2^{2^{i+1}}$ such that $f(r_{i+1}) \equiv 0 \space\mathrm{mod}\space 2^{2^{i+1}}$

$$
\begin{aligned}
r_{i+1} &\equiv r_i - f(r_i) * (f'^{-1}(r_i) \space\mathrm{mod}\space 2^{2^i}) \space\mathrm{mod}\space 2^{2^{i+1}} \\
&\equiv r_i - (z' * r_i - 1)r_i \space\mathrm{mod}\space 2^{2^{i+1}}\\
&\equiv r_i(2 - z' * r_i) \space\mathrm{mod}\space 2^{2^{i+1}}\\
\end{aligned}
$$

We can solve this in $O(lg \space lg \space n)$

To optimize further, $r_2 = 3*z'^2 \space\mathrm{mod}\space 2^4$ [here](https://arxiv.org/abs/1303.0328)

## Note

### Mulmod on large integers

Ref: Russian peasant multiplication

**Goal**: compute $\space\mathrm{mulmod}(x,y,z)\space = (x*y)\space\mathrm{mod}\space z ; \space x,y,z \le 2^{256}-1$

Note: we assume that $x < y$

Example: $(13 * 18) \space\mathrm{mod}\space 19 \equiv X \space\mathrm{mod}\space 19$

Solution:

1. decompose smaller integer into the powers of two.

$$
\begin{aligned}
(13 * 18) \space\mathrm{mod}\space 19 \equiv (2^3 + 2^2 + 2^0) * 18 \space\mathrm{mod}\space 19\\
\end{aligned}
$$

2. calculate the modulo of the multiplication of the larger integer and the powers of two.

$$
\begin{aligned}
F_i &\equiv 2^i*y \space\mathrm{mod}\space z\\
    &\equiv 2 * F_{i-1} \space\mathrm{mod}\space z \\
\end{aligned}
$$

3. sum all $F_i$ that the digit of the index i of the smaller integer in base-2 is 1.

$$
X \equiv \sum{b_i * F_i} \space\mathrm{mod}\space z
$$

$$
b_i = \begin{cases}
    1& \text{if bit at index i of x in base-2 is 1}    \\
    0              & \text{otherwise}
    \end{cases}
$$

to calculate $(a + b)\space\mathrm{mod}\space z$ without overflow.

$$
a + b \space\mathrm{mod}\space z \equiv \begin{cases}
    a+b \space\mathrm{mod}\space z & \text{if } a < z - b\\
    a-(z-b) \space\mathrm{mod}\space z & \text{otherwise}
    \end{cases}
$$

Computation cost: $O(lg\space n)$

### Inverse Modulo

**Goal**: compute $x/y \space\mathrm{mod}\space z \text{ where } \gcd(y,z) = 1 \space\text{and}\space y \mid x$

Example: $((13 * 18) / 6) \space\mathrm{mod}\space 19 \equiv X \space\mathrm{mod}\space 19$

Somehow, we know that $6 \space\mathrm{mod}\space 19 * 16 \space\mathrm{mod}\space 19 \equiv 1  \space\mathrm{mod}\space 19$

$$
\begin{aligned}
((13 * 18)/ 6) * 6 \space\mathrm{mod}\space 19 &\equiv (13 * 18) \space\mathrm{mod}\space 19 \\
((13 * 18) / 6) \space\mathrm{mod}\space 19 * 6 \space\mathrm{mod}\space 19 * 16 \space\mathrm{mod}\space 19&\equiv (13 * 18) \space\mathrm{mod}\space 19 * 16 \space\mathrm{mod}\space 19\\
((13 * 18) / 6) \space\mathrm{mod}\space 19 &\equiv (13 * 18) \space\mathrm{mod}\space 19 * 16 \space\mathrm{mod}\space 19\\
\end{aligned}
$$

But How do we know that we should multiply $16\space\mathrm{mod}\space 19$ to the equation?

**New Goal**: Find $y^{-1} \space\mathrm{mod}\space z$
such that $(y \space\mathrm{mod}\space z) * (y^{-1} \space\mathrm{mod}\space z) \equiv 1 \space\mathrm{mod}\space z$
if $\gcd(y,z) = 1$

Solution:

1. according to the Euler theorem: if $\gcd(y,z) = 1 \space$, $y^{\phi(z)} \space\mathrm{mod}\space z \equiv 1 \space\mathrm{mod}\space z$

$$
y^{-1} \space\mathrm{mod}\space z \equiv y^{\phi(z) - 1} \space\mathrm{mod}\space z
$$

2. Finding $\phi(z)$ (Euler's totient function)

$$
\phi(z) = z\prod_{p|z}{1 - \frac{1}{p}} \space ; \space p  \in\mathbb{P}
$$

For example, $z = 19$

$\phi(19) = 19 * \frac{18}{19} = 18$. So, $6^{-1} \space\mathrm{mod}\space 19 \equiv 6^{18-1} \space\mathrm{mod}\space 19\equiv 16 \space\mathrm{mod}\space 19$

More optimization: Carmichael's theorem $\lambda(n)$:
if $\gcd(y,z) = 1 \space$
, $y^{\lambda(z)} \space\mathrm{mod}\space z \equiv 1 \space\mathrm{mod}\space z$

$$
\lambda(n) = \prod_{p|z}{\lambda(p^{r})} \space ; \space p  \in\mathbb{P}
$$

$$
\lambda(p^r) =
\begin{cases}
\frac{1}{2}\phi(p^r) & \text{if } p = 2 \space\wedge\space r \ge 3 \\
\phi(p^r) &\text{otherwise}
\end{cases}
$$

## References:

1. https://xn--2-umb.com/21/muldiv/index.html
2. https://xn--2-umb.com/18/multiplitcative-inverses/index.html
3. https://en.wikipedia.org/wiki/Carmichael_function
4. http://mathonline.wikidot.com/examples-of-applying-hensel-s-lemma
5. https://en.wikipedia.org/wiki/Hensel%27s_lemma

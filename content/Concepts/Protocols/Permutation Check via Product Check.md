Or equivalently: Prove equality of two multi-sets

Given two length-$m$ lists $A = (a_1, ..., a_n)$ and $B = (b_1, ..., b_n)$, we define:
$$
p_A(X) := \prod_{i = 1}^n(a_i - X)
$$
and
$$
p_B(X) := \prod_{i = 1}^n(b_i - X)
$$
These are equal if and only if $a$ and $b$ are permutations of each other. Equality of polynomials can be checked at a random point $\beta$ due to the [[Schwartz-Zippel Lemma]].

The naive way to compute this would be to have two extra witness columns that compute the accumulated product. However, we can do this with only one extra column, constrained to be equal to:
$$
Z_{2^k} = Z_0 = 1
$$
$$
Z_{i + 1} = Z_i \cdot \frac{(a_i - \beta) }{(b_i - \beta)}
$$
Also, multiple permutation arguments can share the same accumulator (with independent challenges), at the cost of increasing the degree of the constraint.
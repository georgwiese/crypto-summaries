*Source: [[Proofs, Arguments, and Zero-Knowledge]], chapter 10.3.1*

Let $p$ be a univariate polynomial of degree $\le D$ defined over a field $\mathbb{F}$ with a *multiplicative* subgroup $H \subseteq \mathbb{F}$ with $|H| = n$. Then, $\sum_{a \in H} p(a) = 0$ iff. there exists polynomials $h^*$ (degree $\leq D - n$) and $f < n - 1$ such that:
$$
p(X) = h^*(X) \cdot \mathbb{Z}(X) + x \cdot f(X)
$$
where $\mathbb{Z}$ is the *vanishing polynomial* of $H$:
$$
\mathbb{Z}_H(X) := \prod_{a \in H}(X - a) = X^n - 1
$$
-> The prover sends $p$, $h^*$, and $f$
-> The prover requests evaluation proofs for all polynomials at a random point to check the identity
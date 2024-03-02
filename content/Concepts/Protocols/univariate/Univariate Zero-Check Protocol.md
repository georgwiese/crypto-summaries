*Source: [[Proofs, Arguments, and Zero-Knowledge]], lemma 9.3*

Let $p$ be a univariate polynomial of degree $\le D$ defined over a field $\mathbb{F}$. It *vanishes* on a set $H \subseteq \mathbb{F}$ (i.e., $p(h) = 0$ for all $h \in H$) iff. the polynomial $\mathbb{Z}_H(X) := \prod_{\alpha \in H}(X - \alpha)$ divides it. That is, if there exists a polynomial $h^*$ of degree $\le d - |H|$ such that $p = \mathbb{Z}_H \cdot h^*$.

If $H$ is a multiplicative subgroup of $\mathbb{F}$ with $|H| = n$, we have
$$
\mathbb{Z}_H(X) := \prod_{a \in H}(X - a) = X^n - 1
$$

So, a polynomial IOP works like this:
- The prover sends $p$ and $h^*$ (whose degrees are bounded)
- The verifier queries both at a random point $r \in \mathbb{F}$ and also evaluates $\mathbb{Z}_H(r)$
- The verifier checks whether $p(r) = \mathbb{Z}_H(r) \cdot h^*(r)$ holds
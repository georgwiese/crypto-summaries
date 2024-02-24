Let $\widetilde{W}$ be a multilinear polynomial over $\mathbb{F}$ with $\log n$ variables. The verifier wishes to evaluate it at two points $b, c \in \mathbb{F}^{\log n}$. The following protocol reduces this to an evaluation at a random point $r \in \mathbb{F}^{\log n}$:
- Let $l: \mathbb{F} \rightarrow \mathbb{F}^{\log n}$ be some canonical line passing through $b$ and $c$, e.g. $l(0) = b$ and $l(1) = c$
- The prover sends a polynomial $q$ claimed to equal $\tilde{W} \circ l$, a *univariate* polynomial of degree at most $\log n$
- -> The prover claims that $\tilde{W}(b) = q(0)$ and $\tilde{W}(c) = q(1)$
- The verifier pics a random point $r^* \in \mathbb{F}$ and sets $r = l(r^*)$, accepts if $q(r^*) = \tilde{W}(r)$

This method generalizes to more than two points, with the degree of $l$ growing linearly with the number of points.
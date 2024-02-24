*Source: [[Proofs, Arguments, and Zero-Knowledge]], page 125*

**Goal**: Given a multivariate polynomial $g: \mathbb{F}^k \rightarrow \mathbb{F}$, check that it vanishes on the *boolean hypercube* (i.e., all inputs in ${0, 1}^k$).

Consider the function $\beta_k: \{0, 1\}^k \times \{0, 1\}^k \rightarrow \{0, 1\}$:

$$
\beta_k(x, y) :=
\begin{cases}
1 & \text{if } x = y, \\
0 & \text{otherwise}.
\end{cases}
$$
It's multilinear extension is:
$$
\widetilde{\beta}_{k}(a, b) = \prod_{j=1}^{k} ((1 - a_j)(1 - b_j) + a_jb_j)
$$
Now, consider:
$$
p(X) := \sum_{u \in \{0, 1\}^k} \widetilde{\beta}_k(X, u) \cdot g(u)
$$
This is a *multilinear* polynomial (even if $g$ is not!) which is identically zero iff. $g$ vanishes on the boolean hypercube. Due to the [[Schwartz-Zippel Lemma]], this can be checked probabilistically by evaluating $p$ at a random point $r \in \mathbb{F}^k$.

Evaluating $p(r)$ can be done by applying the [[Multivariate Sum-Check Protocol]] to the polynomial:
$$
h(Y) := \beta_k(r, Y) \cdot g(Y)
$$

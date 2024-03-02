- **Terminology**: Given an $m$-variate polynomial $g$
	- The **degree of a term** is the sum of the exponents in the term
	- The **total degree** of a $g$ is the maximum of the degrees of the term

Let $\mathbb{F}$ be any field, and let $g: \mathbb{F}^m \rightarrow \mathbb{F}$ be a nonzero $m$-variate polynomial of total degree at most $d$. Then on any finite set $S \subseteq \mathbb{F}$,

$$
\text{Pr}_{x \leftarrow S^m} [g(x) = 0] \leq \frac{d}{|S|}.
$$

Here, $x \leftarrow S^m$ denotes an $x$ drawn uniformly at random from the product set $S^m$, and $|S|$ denotes the size of $S$.

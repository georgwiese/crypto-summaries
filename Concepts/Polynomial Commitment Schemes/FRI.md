*Source: [[Proofs, Arguments, and Zero-Knowledge]], section 10.4*

FRI let's the prover specify a function $g: L_0 \rightarrow \mathbb{F}$ where $L_0 \subset \mathbb{F}$ and $g$ is claimed to be a polynomial of degree at most $d$ (i.e., $g$ is a *codeword in the Reed-Solomon code*). $|L_0| = \rho^{-1} \cdot d$, where $0 < \rho < 1$ is the *rate parameter* (trades off prover time and proof size).

The general idea:
- In order to show that a polynomial $G_i(z)$ is at most of degree $d_i$, show that it is equal to $Q_i(z, z^2)$ for a *bivariate* polynomial $Q_i(Z, Y)$ of degree $1$ in $Z$ and $d_i / 2$ in $Y$.
- $Q_i(z, y) := P_{i, 0}(y) + z \cdot P_{i, 1}(y)$ where $P_{i, 0}$ (respectively, $P_{i, 1}$) consist of all monomials of $G_i$ of even (respectively, odd) degree, but with all powers divided by two and then replaced by their integer floor.
- -> $Q_i(r, Y)$ (for a randomly picked $r \in \mathbb{F}$) is univariate polynomial of degree $d_{i + 1} = d_i / 2$
- -> Recurse!

**Commitment Phase**:
For each round $0 \leq i \leq \log_2(d)$, the verifier picks a random value $x_i \in \mathbb{F}$ and requests the polynomial:
$$
G_{i + 1}(Y) := Q(x_i, Y) = P_{i, 0}(Y) + x_i \cdot P_{i, 1}(Y)
$$
In each round, the polynomial is defined over the domain $L_i$ with $L_{i + 1} = \{x^2 | x \in L_i\}$. $L_0$ is chosen to be a multiplicative subgroup of $\mathbb{F}$ of a power of $2$, which implies that $|L_{i + 1}| = |L_i| / 2$.

**Query Phase** (repeated $l$ times):
The verifier picks a random $s_0 \in L_0$ and sets $s_{i + 1} = s_i^2$. Now, it needs to check that $G_{i + 1}(s_{i + 1}) = Q(x_i, s_{s + 1})$.
The verifier queries $G_i(s_i)$ and $G_i(s'_i)$ for $s'_i = - s_i$ (note that $s_{i + 1} = s_i^2 = (s'_i)^2$). From that and $x_i$, it is possible to compute $G_{i + 1}(s_{i + 1})$, assuming the relationship is as claimed.

*For details, refer to the book, I didn't quite understand them.*

**Queries to points outside $L_0$**:
$p(r) = v$ is equivalent to the assertion that there exists a polynomial $w$ of degree $d - 1$ such that:
$$
p(X) - v = w(X) \cdot (X - r) 
$$
-> The verifier can apply FRI to $(p(X) - v) \cdot (X - r)^{-1}$, which can be evaluated at any point in $L_0$ by querying $p$.
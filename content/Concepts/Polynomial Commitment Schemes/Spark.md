*Sources:*
- [[Brakedown - Linear-time and field-agnostic SNARKs for R1CS]]
- [[Unlocking the lookup singularity with Lasso]]
- (Originally described as part of [[Spartan]])

**Use-case**: In [[Spartan]], we need to commit to the *sparse* R1CS matrices, which are of size $N = M^2$, but only have $m \in O(M)$ non-zero entries. They achieve a quadratic speed-up over using a dense commitment scheme.

**Representing sparse polynomials with dense polynomials**: Any $(\log N)$-variate multilinear polynomial $D$ can be written as:
$$
D(r) = \sum_{i \in \{0, 1\}^{\log N}: D(i) \neq 0} D(i, j) \cdot \widetilde{eq}_{\log N}(i, r)
$$
where $\widetilde{eq}_s(x, y)$ is the unique multi-linear extension of the equality function on the boolean hypercube:
$$
\widetilde{eq}_s(x, y) = \prod_{i = 1}^s(x_i y_i + (1 - x_i)(1 - y_i))
$$
 
 $\widetilde{eq}_{\log N}(i, \cdot)$ is the $i^{th}$ Lagrange polynomial on $\{0, 1\}^N$. We can factor it into two Langrange polynomials over $\{0, 1\}^M$:
$$
D(r_x, r_y) = \sum_{i \in \{0, 1\}^{\log M}, j \in \{0, 1\}^{\log M}: D(i, j) \neq 0} D(i, j) \cdot \widetilde{eq}_{\log M}(i, r_x) \cdot \widetilde{eq}_{\log M}(j, r_y)
$$

Let $\mathtt{bits}: \mathbb{F} \rightarrow \mathbb{F}^{\log N}$  be the function that maps a field element to its bit representation. Then, we can write:
$$
D(r_x, r_y) = \sum_{k \in \{0, 1\}^{\log m}} val(k) \cdot \widetilde{eq}(\mathtt{bits}(row(k)), r_x) \cdot \widetilde{eq}(\mathtt{bits}(col(k)), r_y)
$$
for some $(\log m)$-variate polynomials $val$, $row$, and $col$.

**Commitment phase**: The prover commits to dense polynomials $val$, $row$, and $col$.

**Evaluation proof**:
- The prover sends the claimed evaluation $v$ of $D(r_x, r_y)$
- The prover commits to two $(\log m)$-variate helper polynomials $E_x$ and $E_y$, such that:
	- $\forall k \in \{0, 1\}^{\log m}: E_x(k) = \widetilde{eq}(\mathtt{bits}(row(k)), r_x)$
	- $\forall k \in \{0, 1\}^{\log m}: E_y(k) = \widetilde{eq}(\mathtt{bits}(col(k)), r_y)$
- Prover and verifier run the [[Multivariate Sum-Check Protocol]] to check that:
  $\sum_{k \in \{0, 1\}^{\log m}} val(k) \cdot E_x(k) \cdot E_y(k)$
- To show that $E_x$ and $E_y$ are well-formed, the prover and verifier run the an interactive protocol derived from the [[Offline Memory Checking]] technique.
	- For example, to show that $E_x$ is well-formed, we show that it corresponds to the vector obtained by reading from a memory described by  $\widetilde{t}(x_1, ..., x_{\log m}) = \widetilde{eq}(x_1, ..., x_{\log m}, r_x)$ using addresses described by $row$.

**Generalizing**: The reason we factored the Lagrange polynomials over $\{0, 1\}^N$ into **two** Lagrange polynomials over $\{0, 1\}^M$ is because $m \in \Theta(M) = \Theta(N^2)$. In general, we could split it into any number $c$ of polynomials where $\log N = c \cdot \log M$.
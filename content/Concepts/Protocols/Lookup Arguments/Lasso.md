*Source: [[Unlocking the lookup singularity with Lasso]]*

### Setting
We have a vectors $t \in \mathbb{F}^N$ and $a \in \mathbb{F}^m$. We want to show that element in $a$ is in $t$. This is equivalent to showing that there exists a matrix $M \in \mathbb{F}^{m \times N}$ whose rows consist of $m$ unit vectors, such that
$$
a = M \cdot t
$$
Up to negligible soundness error, this is equivalent to:
$$
\widetilde{a}(r) = \sum_{y \in \{0, 1\}^{\log N}} \widetilde{M}(r, y) \cdot \widetilde{t}(r)
$$
where $\widetilde{a}$, $\widetilde{t}$, and $\widetilde{M}$ are the multilinear extensions and $r$ is a random verifier challenge.

### Starting Point: Spark
We commit to $\widetilde{M}$ using [[Spark]]. However, we can specialize it such that:
- The scheme becomes concretely more efficient
- It automatically enforces that rows are unit vectors

Recall that in the standard Spark evaluation proof, we would show that:
$$
\widetilde{M}(r_x, r_y) = \sum_{k \in \{0, 1\}^{\log m}} val(k) \cdot \widetilde{eq}_{\log N}(\mathtt{bits}(row(k)), r_x) \cdot \widetilde{eq}_{\log m}(\mathtt{bits}(col(k)), r_y)
$$
where commitments to $val$, $row$ and $col$ represent the commitment to $\widetilde{M}$. But because $M$'s rows should be unit vectors we have:
- $val(k) = 1$, so it can be removed
- $\mathtt{bits}(row(k)) = k$, which means we have one less helper polynomial to commit to

Also, Lasso uses the general version of Spark, which factors the Lagrange polynomials into $c$ parts, so the prover commits to $c$ $((\log n) / c)$-variate polynomials $dim_1, ..., dim_c$ and shows that:
$$
\widetilde{M}(r_x, r_y^{(1)}, ..., r_y^{(c)}) = \sum_{k \in \{0, 1\}^{\log m}} \widetilde{eq}(k, r_x) \cdot \prod_{i = 1}^c \widetilde{eq}_{\log N/c}(\mathtt{bits}(dim_i(k)), r_y^{(i)})
$$
### Surge: A generalization of Spark, providing Lasso

Surge generalizes Spark to directly prove:
$$
\sum_{y \in \{0, 1\}^{\log N}} \widetilde{M}(r, y) \cdot \widetilde{t}(r)
$$
for some $r \in \mathbb{F}^{\log m}$ and multi-linear extension $\widetilde{t}$ of a table $T$.

We assume that the table has a **Spark-only structure (SOS)** (or: is **decomposable**), which means that each entry $T[i]$ (with $i \in \{1, ..., N\}$) can be evaluated quickly by consulting $\alpha \in O(c)$ subtables $T_1, ..., T_{\alpha}$ of size $N^{1 / c}$.
To evaluate $\widetilde{t}(r)$ with $r \in \{0, 1\}^{\log N}$, write $r = (r_1, ..., r_c) \in (\{0, 1\}^{\log N / c})^c$. Then, there should be an $\alpha$-variate polynomial $g$ such that:
$$
\forall r \in \{0, 1\}^{\log N}: T[r] = g(T_1[r_1], ..., T_{k}[r_1], ..., T_{\alpha - k + 1}[r_c], ..., T_{\alpha}[r_c])
$$
where $k = \alpha / c$.

The rest is a straight-forward generalization of Spark:
- The commitment to $\widetilde{M}$ consists of commitments to $c$ $(\log m)$-variate dense polynomials $dim_1, ..., dim_c$
- For evaluation, the prover commits to $(\log m)$-variate helper polynomials $E_1, ..., E_{\alpha}$
- To show that $v = \sum_{y \in \{0, 1\}^{\log N}} \widetilde{M}(r, y) \cdot \widetilde{t}(r)$, the [[Multivariate Sum-Check Protocol]] is applied to polynomial
  $h(k) := \widetilde{eq}(r, k) \cdot g(E_1(k), ..., E_{\alpha}(k))$
- Finally, the prover shows that $E_1, ..., E_{\alpha}$ are well-formed using [[Offline Memory Checking]] (which requires it to send commitments to $m + N^{1 / c}$ read counts for each)
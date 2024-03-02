*Source: [[Proofs, Arguments, and Zero-Knowledge]], Section 14.2*

A generalized version of [[Pedersen Commitment]] that allows the prover to commit to a vector $u$.
Let $g_1, ..., g_n$ and $h$ be randomly chosen generators for $\mathbb{G}$. Also, let $r_u \in \{0, 1, ..., |\mathbb{G}| - 1\}$ be randomly chosen. Then:
$$
Com(u; r_u) := h^{r_u} \cdot \prod_{i = 1}^n g_i^{u_i}
$$
#### Polynomial Commitment Scheme
To commit to a polynomial, simply use the scheme above for the coefficient vector. The commitment has *constant size*.

For evaluation (*linear time*), follow this protocol:
![[generalized_pedersen_eval_proof.png]]
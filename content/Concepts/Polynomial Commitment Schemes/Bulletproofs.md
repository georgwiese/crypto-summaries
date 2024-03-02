*Source: [[Proofs, Arguments, and Zero-Knowledge]], Section 14.4*

This scheme achieves:
- *Constant* commitment size
- ***Logarithmic*** evaluation proof size
- *Linear* verifier time
### Key idea

The commitment is a [[Generalized Pedersen Commitment]] (but without the blinding factor). However, in this section, we use *additive notation*, so a commitment can be seen as a *dot product of the coefficient vector $u$ with a vector of group elements $\mathbf{g}$*:
$$
Com(z) = \langle u, \mathbf{g} \rangle = \sum_{i = 1}^n u_i \cdot g_i
$$
Let's break the vectors into two halves, $u = u_L \circ u_R$ and $\mathbf{g} = \mathbf{g}_L \circ \mathbf{g}_R$ (where $\circ$ denotes concatenation). Note that $\langle u, \mathbf{g} \rangle = \langle u_L, \mathbf{g}_L \rangle + \langle u_R, \mathbf{g}_R \rangle$.

Now, for a randomly picked $\alpha$, let:
$$
\begin{align*}
u' &= \alpha \cdot u_L + \alpha^{-1} \cdot u_R \\
\mathbf{g}' &= \alpha \cdot \mathbf{g}_L + \alpha^{-1} \cdot \mathbf{g}_R
\end{align*}
$$
This cuts the length of $u$ and $\mathbf{g}$ in half. Their dot products are related as follows:
$$
\langle u', \mathbf{g}' \rangle = \langle u, \mathbf{g} \rangle + \alpha^2 \langle u_L, \mathbf{g}_R \rangle + \alpha^{-2} \langle u_R, \mathbf{g}_L \rangle
$$
With these ideas, one can build a protocol that let's the prover prove that it knows an opening to the commitment:
- Prover sends the commitment $c =\langle u, \mathbf{g} \rangle$
- For $\log n$ rounds:
	- Prover sends $v_L, v_R$ claimed to equal $\langle u_L, \mathbf{g}_R \rangle$ and $\langle u_R, \mathbf{g}_L \rangle$
	- The verifier picks a random challenge $\alpha$ and sets $c := c + \alpha^2 v_L + \alpha^{-2} v_R$ and $\mathbf{g} = \alpha \cdot \mathbf{g}_L + \alpha^{-1} \cdot \mathbf{g}_R$
- Now, there is a derived "vector" $u$ of size $1$ (which the prover can compute) such that $c =\langle u, \mathbf{g} \rangle$. The prover sends $u$ and the verifier checks the equality
### The polynomial commitment scheme

What we're actually interested in is $\langle u, y \rangle$ for $y = (1, z, z^2, ..., z^{n - 1})$. The idea is to run the protocol in *parallel* for both $\langle u, \mathbf{g} \rangle$ and $\langle u, y \rangle$. Note that $\mathbf{g}$ is a vector of **group elements** and $y$ is a vector of **field elements**, but since both support addition, we can perform the same calculations on them.
### The algorithm

![[bulletproofs.png]]

### Zero Knowledge

Using *commit-and-prove* style techniques, the prover can send hiding [[Pedersen Commitment]]s and prove in zero knowledge that the committed values do pass the final verifier check.

### Relation to other commitment schemes

Compared to [[Hyrax]], verification time is worse ($O(n)$ vs. $O(\sqrt{n})$) but evaluation proof size is better. Both ideas can be combined which would:
- Increases the commitment size from $O(1)$ to $O(\sqrt{n})$
- Reduces the public parameter size and verifier runtime from $O(n)$ to $O(\sqrt{n})$
- Maintains the $O(\log n)$ evaluation proof size

[[Dory]] reduces the verifier runtime from $O(n)$ to $O(\log n)$ but requires pairings.
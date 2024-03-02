*Source: [[Proofs, Arguments, and Zero-Knowledge]], section 15.4*

Cost summary:
- $O(\sqrt{n})$ pre-processing time for a *transparent* setup (i.e., no toxic waste)
- $O(\log n)$ proof size (although concretely larger than [[Bulletproofs]])
- $O(\log n)$ verification time

Uses [[Inner-pairing-product commitment]]s, denoting both $\mathbb{G}_1$ and $\mathbb{G}_2$ as $\mathbb{G}$ **even though they must not equal each other** for DDH to hold and the scheme to be binding.

**Why is the verier time $O(n)$ in [[Bulletproofs]]?** Because the following update takes $O(n)$ time: $\mathbf{g}' = \alpha \cdot \mathbf{g}_L + \alpha^{-1} \cdot \mathbf{g}_R$

**Pre-processing procedure:**
- In round 1, commit to $g_L^{(0)}$ and $g_R^{(0)}$ using *public*, randomly chosen commitment key $\Gamma^{(1)} \in \mathbb{G}^{n / 2}$: $\Delta_L^{(1)} = \langle g_L^{(0)}, \Gamma^{(1)} \rangle$ and $\Delta_R^{(1)} = \langle g_R^{(0)}, \Gamma^{(1)} \rangle$
- In round $i$, write $\Gamma^{(i - 1)}$ as $\Gamma^{(i - 1)}_L$ and $\Gamma^{(i - 1)}_R$ and commit to both using a new commitment key $\Gamma^{(i)} \in \mathbb{G}^{n / 2^i}$

**A knowledge of opening protocol ($O(\log^2 n)$ proof size & verifier time)**: Like Bulletproofs, except instead of updating $\mathbf{g}$, the verifier computes a commitment to it in constant time, using homomorphism of the commitment:
$$
c_{\mathbf{g}^{(i)}} = \alpha_i^{-1} \Delta_L^{(i)} + \alpha_i \Delta_R^{(i)}
$$
So, in addition to proving that the prover knows $u_{(i)}$ such that $c_{u^{(i)}} = \langle u^{(i)}, \mathbf{g}^{(i)} \rangle$, the prover also shows that (s)he knows $\mathbf{g}_{(i)}$ such that $c_{\mathbf{g}^{(i)}} = \langle \mathbf{g}^{(i)}, \Gamma^{(i)} \rangle$. This can be done via recursion by executing the protocol in parallel, using the same randomness.
-> Because the number of vectors involved in each round increases by one every round, the total proof size & verifier time is $O(\log^2 n)$

**Extending to a polynomial commitment**: Analogous to [[Bulletproofs]]

**Reducing pre-processing time to $O(\sqrt{n})$ via matrix commitments**: One can apply the same trick as in [[Hyrax]], except the size of the commitment stays the same. The trick is that the prover can compute [[Generalized Pedersen Commitment]] "in its own head" and then compute an [[Inner-pairing-product commitment]] of the vector of commitments.

**Reducing to $O(\log n)$ proof size and verification time**: The key idea is to double the rounds of interaction to $2 \log n$ in order to keep the messages per round constant. For details refer to the book.

The final protocol:
![[dory.png]]
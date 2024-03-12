*Sources:*
- [[Proofs, Arguments, and Zero-Knowledge]], section 10.5
- [[Brakedown - Linear-time and field-agnostic SNARKs for R1CS]]

These protocols exploit the [[Tensor Product Structure in Polynomial Evaluation Queries]] to commit to a degree-$n$ polynomial $q$. Let $m := \sqrt{n}$.

Ligero and Brakedown use a different [[Error-correcting code]] $E$ with rate $\rho$:
- **Ligero** uses the univariate low-degree extension code -> $O(\rho^{-1} m \cdot \log(\rho^{-1} m))$
- **Brakedown** uses a code with a **linear-time** encoding procedure -> $O(\rho^{-1} m)$

**Commitment Phase**:
To commit to a polynomial $q$ with coefficient vector $u$, let's view $u$ as an $m \times m$ matrix and let $\hat{u}$ be the $m \times (\rho^{-1} m)$ matrix where each row is replaced with $E(u_i)$, then:
- The prover sends a commitment to matrix $M$, claimed to equal $\hat{u}$

The verifier has to check that $M$ is well-formed, i.e., every row is a valid code word: 
- The verifier chooses a random vector $r \in \mathbb{F}^m$
- The prover responds with $w := r^T \cdot M$ (which must be a codeword, so the prover can just send $v$ such that the result is $E(v)$)
	- *In other words, the prover requests a random linear combination of rows.*
- For a random column subset of size $t = \Theta(\lambda)$, the verifier checks entries of $w$ by opening the entire column from the commitment of $M$ and re-computes it

**Evaluation Phase**:
 $z$ means computing $b^T \cdot u \cdot a$ for $a := (1, z, z^2, ..., z^{m - 1})$ and $b := (1, z^m, z^{2m}, ..., z^{m (m - 1)})$.

Computing $b^T \cdot u$ works completely analogous to the "random linear combination test". Then, $q(z) = \langle w, a \rangle$ can be computed in $O(m)$ time.

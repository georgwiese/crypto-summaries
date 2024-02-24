*Source: [[Proofs, Arguments, and Zero-Knowledge]], Section 14.3*

Achieves $O(\sqrt{n})$ commitments and $O(\sqrt{n})$ evaluation proofs. Let $m := \sqrt{n}$.
- Instead of committing to the *vector* of coefficients, the prover views the coefficients as a $m \times m$ matrix and commits to *each column of the matrix*.
- As explained in [[Tensor Product Structure in Polynomial Evaluation Queries]], an evaluation can be viewed as computing $b^T \cdot u \cdot a$, using to size-$m$ vectors $a, b$, known to the verifier.
- The verifier can compute the commitment to $u \cdot a$ on its own as $\prod_{j = 1}^m Com(u_j)^{a_j}$ using additive homomorphism. Then, the prover provides an evaluation proof of $\langle b, u \cdot a \rangle$ which is the same as that of [[Generalized Pedersen Commitment]].
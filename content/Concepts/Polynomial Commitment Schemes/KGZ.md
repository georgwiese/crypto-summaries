*Source: [[Proofs, Arguments, and Zero-Knowledge]], section 15.2*
*Also, [this write-up](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html) is excellent and includes batching techniques.*

### The Scheme
**Powers-of-Tau Structured Reference String (SRS)**: $(g, g^\tau, g^{\tau^2}, ..., g^{\tau^{n - 1}})$ for a randomly chosen $\tau$ (which is *toxic waste*).
-> Anyone can calculate $g^{q(\tau)}$ for any degree-$n$ polynomial $q$ *without knowing $\tau$*!

**Commitment to $q$**: $c = g^{q(\tau)}$.

**Evaluation proof of $q(z) = v$**: $y = g^{w(\tau)}$ where
$$
w(X) := (q(X) - v) / (X - z)
$$
-> This polynomial exists iff. $q(z) = v$.

**Verifier check**: Using *pairing* $e$, the verifier checks that
$$
e(c \cdot g^{-v}, g) = e(y, g^\tau \cdot g^{-z})
$$

### Cryptographic Assumptions
- **The D-strong Diffie-Hellman assumption**: Dividing by $\tau - z$ in the exponent is intractable given the SRS
	- Implies binding property, but not extractability!
- **Generic Group Model (GGM)**: The adversary can only perform the group and pairing operation by querying an oracle
	- Implies extractability
- **Algebraic Group Model (AGM)**: Given some known group elements $L_1, ..., L_t$, any efficient algorithm that outputs a group element $g$ also outputs numbers $c_1, ..., c_t$ such that $g = \prod_{i = 1}^t L_i^{c_i}$ 
	- Weaker than GGM
	- Also implies extractability
### Extension to Multilinear Polynomials
An extension due to PST13, which exploits the fact that for any fixed $z = (z_1, .., z_l) \in \mathbb{F}_p^l$, $q(z) = v$ iff. there exists multilinear polynomials $w_1, ..., w_l$ such that:
$$
q(X) - v = \sum_{i = 1}^l (x_i - z_i)  \cdot w_i(X)
$$
With this fact, KZG commitments can be modified as follows:
- The SRS contains all Lagrange basis polynomials evaluated at a random point $r \in \mathbb{F}^l$
- The prover commits to all $l$ witness polynomials
- The verifier checks that $e(c \cdot g^{-v}, g) = \prod_{i = 1}^l e(y_i, g^{r_i} \cdot g^{-z_i})$

Evaluation proof size and verifier runtime increase to $O(l)$.
*Source: [[Proofs, Arguments, and Zero-Knowledge]], Section 17.2*

**Linear Function**:
- A function $\pi: \mathbb{F}^v \rightarrow \mathbb{F}$ for some large $v$
- $\pi(d_1q_1 + d_2q_2) = d_1\pi(q_1) + d_2\pi(q_2)$
- Equivalently, $\pi$ is a **$v$-variate polynomial of total degree $1$ with a constant term of $0$**.

The main challenge is that we want the prover's work to be $O(v)$, not $O(|\mathbb{F}|^v)$.

**Commitment scheme**: Makes use of a *semantically secure, additively homomorphic encryption scheme (e.g. [ElGamal encryption](https://en.wikipedia.org/wiki/ElGamal_encryption))*

- **Commitment phase**:
	- The verifier chooses a vector $r = (r_1, ..., r_v) \in \mathbb{F}^v$ at random and sends an encryption to the prover
	- The prover computes the encryption of $s = \pi(r) = \langle d, r \rangle$ for some vector $d$ (since $\pi$ is linear), exploiting additive homomorphism of the encryption scheme
- **Reveal Phase**: Let $q^{(1)}, ..., q^{(k)} \in \mathbb{F}^v$ be $k$ query vectors
	- The verifier chooses $\alpha_1, ..., \alpha_k \in \mathbb{F}$ at random and sends $q^{(1)}, ..., q^{(k)}$ as well as $q^* = r + \sum_{i = 1}^k \alpha_i \cdot q^{(i)}$
	- The prover responds with $a^{(1)}, ..., a^{(k)}, a^*$, claimed to be the evaluations at those points
	- The verifier approves if $a^* = s + \sum_{i = 1}^k \alpha_i \cdot a^{(i)}$
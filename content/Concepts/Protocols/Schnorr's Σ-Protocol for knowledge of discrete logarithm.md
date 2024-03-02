*Source: [[Proofs, Arguments, and Zero-Knowledge]], Chapter 12*

- Given a generator $g$, the **discrete log relation** is the collection of pairs $(h, w) \in \mathbb{G} \times \mathbb{Z}$ such that $h = g^w$
- **$\Sigma$-Protocol**: A 3-message public coin protocol that convinces the verifier that the prover knows a tuple $(h, w)$ in the relation for a public $h$
	- Let's call the messages $(a, e, z)$, where $a$ and $z$ are prover messages and $e$ is a random challenge
- **The protocol**:
	- $a$: The prover picks a random number $r$ and sends $g^r$ to the verifier
	- $e$: The verifier responds with a random challenge $e$
	- $z$: The prover responds with $(we + r)$
	- -> The verifier checks that $a \cdot h^e = g^z$
- **Honest-verifier perfect zero-knowledge (HVZK)**: There is a simulator that can generate transcripts of *identical* distribution
	- The simulator generates $e$ and $z$ randomly, then solves for $a$
- **Special soundness**: There exists a polynomial-time algorithm that for any pair of distinct accepting transcripts $(a, e, z)$ and $(a, e', z')$ can output a witness $(h, w)$
	- If the prover was prepared to answer to more than one challenge, (s)he could have run that algorithm to get the witness, so (s)he must know it!
	- In this case, setting $h = g^{w^*}$, we can solve for $w^* = (z - z') \cdot (e - e')^{-1}$
- **Fiat-Shamit transformation**: Set $e = H(h, a)$, where $H$ is modeled as a random oracle
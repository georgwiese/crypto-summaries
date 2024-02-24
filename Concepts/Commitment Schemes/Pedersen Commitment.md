*Source: [[Proofs, Arguments, and Zero-Knowledge]], Chapter 12*

## Turning $\Sigma$-protocols into perfectly hiding commitment schemes
- **Special HVZK**: The simulator must be able to generate an accepting transcript for any concrete challenge $e^*$
	- True for [[Schnorr's Σ-Protocol for knowledge of discrete logarithm]]
- **Protocol**:
	- **Key generation**: Generate a random element $(h, w)$ in the relation
		- $h$ is both the committing and verification key
		- $w$ is considered toxic waste that must be discarded
			- Note that if the relation is discrete log, $h$ is just a random group element and can be generated *transparently* (i.e., without producing toxic waste)
	- **Commitment**: For a message $m$, set $e^* = m$ and run the simulator to generate an accepting transcript $(a, m, z)$ -> $a$ is the commitment
		- For Schnorr's protocol: $a = g^z \cdot (h^m)^{-1}$
		- Perfectly hiding if the $\Sigma$-protocol has perfect zero knowledge, because $a$ is independent $m$ in the "normal" run of the $\Sigma$-protocol
	- **Opening**: The committer sends $m$ and $z$; the verifier checks that $(e, m, z)$ is accepting
		- Computational binding property follows from special soundness!
## Pedersen Commitment
Equivalent to the previous transformation applied to [[Schnorr's Σ-Protocol for knowledge of discrete logarithm]], but usually formulated differently:
- $g$ and $h$ are two random generators *for which the discrete log is unknown*
- The commitment is $c = g^m \cdot h^z$ for a random $z$ (called a *blinding factor*)
- The verifier checks this relationship when given $m$ and $z$
- -> **Additively homomorphic**: Given commitments to messages $m_1$ & $m_2$, one can compute a commitment to the message $m_1 + m_2$
- **Perfect HVZK proof of Knowledge of an opening**: Let's a prover prove that (s)he *knows* and opening to a Pedersen commitment, without actually opening it:
	- The prover sends $a = Com(d, r)$ for two random elements $d, r$
	- The verifier sends a challenge $e$ and derives the commitment to $me + d$
	- The prover opens this commitment
- **Establishing a product relationship between committed values**:
	- While Pedersen commitments are additively homomorphic, they are not multiplicatively homomorphic
	- But the prover can provide $c_i = Com(m_i, r_i)$ for $i \in \{1, 2, 3\}$ and prove in zero knowledge that $m_3 = m_1 \cdot m_2$
	- The trick is that if this is true, the commitment to $m_3$ using generators $g$ and $h$ can be seen as a commitment to $m_2$ using generators $c_1$ and $h$
	- *For details, refer to the book*
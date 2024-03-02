*Source: [[Proofs, Arguments, and Zero-Knowledge]], Section 18.5*



#### Relaxed R1CS
We'll prove knowledge of a witness for a *relaxed* R1CS instance.

We'll have the following **public** values:
- Matrices $A, B, C \in \mathbb{F}^{n \times n}$
- Vectors $s := (y_{i - 1}, y_i) \in \mathbb{F}^m$
- Scalar $u \in \mathbb{F}$

An the following **committed** values:
- Witness vector $w \in \mathbb{F}^{n -m}$
- Slack vector $E \in \mathbb{F}^n$

Let $s := (y_{i - 1}, y_i)$, $z := (s, w) \in \mathbb{F}^n$. We want to make the claim that:
$$
(A \cdot z) \circ (B \cdot z) = u \cdot (C \cdot z) + E
$$
#### Folding two instances
Let $r \in \mathbb{F}$ be randomly picked by the verifier. Then, we set:
$$
s := s_1 + r \cdot s_2
$$
$$
w := w_1 + r \cdot w_2
$$
$$
u := u_1 + r \cdot u_2
$$
$$
E := E_1 + r^2 \cdot E_2 + r \cdot T
$$
Where $T$ are the cross-terms:
$$
T := (A \cdot z_2) \circ (B \cdot z_1) + (A \cdot z_1) \circ (B \cdot z_2) - u_1 \cdot C \cdot z_2 - u_2 \cdot C \cdot z_1
$$
-> The result is also a valid relaxed R1CS instance!
-> The prover can commit to $T$, $E$, and $w$ using a *homomorphic* vector commitment, then the validator samples $r$
#### A Large non-interactive Argument
For each application of the circuit, the verifier keeps track of:
- The current round number
- The values $y_{j - 1}$ and $y_j$ (determining $s_j$)
- The values $u_j$
- $c_w^{(j)}$: The commitment to $w_j$
- $c_E^{(j)}$: The commitment to $E_j$

The protocol proceeds as follows:
- In each round, the prover sends $y_j$, $c_w^{(j)}$, and $c_T^{(j)}$
- In round 1, the verifier sends the round number to 1, $u_1 := 1$, $y_0 := x$ (the input to the incremental computation), and $c_E^{(j)}$ to a commitment to the vectors of all $0$s
- Otherwise, the verifier increments the round number and updates the running instance as described above
- In the final round, the prover also outputs a SNARK for the folded instance, e.g. using a generalization of [[Spartan]] with [[Bulletproofs]]

The proof is the concatenation of all the values kept track of by the verifier + the final SNARK.
#### Proving the folding
Idea: Augment the circuit to do the verifier's work in each step:
- All verifier's variables become *public inputs*
- All prover inputs become *private input* (except $y_j$, which the circuit already computes)
- The new values of the verifier's variables become *public output*
	- *Question: How do we make sure these outputs are actually used as inputs in the next step?*

The extra work that has to be computed in the circuit amounts to:
- Invoking a cryptographic hash function to obtain $r$ via the Fiat-Shamir transformation
- A few field multiplications and additions over $\mathbb{F}_p$
- Computing the updates of the homomorphic commitments
	- Dominates the recursion overhead if the hash function is SNARK-friendly
	- In the case of [[Pedersen Commitment]]s: One group exponentiation and one group multiplication for each of the two commitments

One needs a cycle of curves, but they do not need to be pairing-friendly.
-> The original function needs to be efficiently computable over *both* fields!
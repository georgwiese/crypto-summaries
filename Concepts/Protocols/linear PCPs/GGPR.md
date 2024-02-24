*Source: [[Proofs, Arguments, and Zero-Knowledge]], Section 17.4*

A linear PCP, for satisfiability of an R1CS instance (in this presentation), i.e., there is a witness $z \in \mathbb{F}^S$ such that
$$
Az \circ Bz = Cz
$$
where $A, B, C \in \mathbb{F}^{l \times S}$ are fixed matrices.

Let $H := \{\sigma_1, ..., \sigma_l\}$ be a set of elements in $\mathbb{F}$ (for example, a multiplicative subgroup). Then, we can interpolate univariate polynomials $A_j, B_j, C_j$ that map $\sigma_i$ to entry $(i, j)$ of the respective matrix. In other words, $M_j$ interpolates column $j$ of matrix $M$.

Then, the following polynomial is zero on all $\sigma_i \in H$ iff. $z$ is a valid witness:
$$
g_z(t) := \left(\sum_{j \in \{1, ..., S\}} z_j \cdot A_j(t)\right) \cdot \left(\sum_{j \in \{1, ..., S\}} z_j \cdot B_j(t)\right) - \left(\sum_{j \in \{1, ..., S\}} z_j \cdot C_j(t)\right)
$$
This can be validated using the [[Univariate Zero-Check Protocol]], which requires the prover to commit to a univariate polynomial $h^*$ such that $g_z = \mathbb{Z}_H \cdot h^*$. The verifier than asks for an evaluation of both at a random point $r$.

The prover commits to linear functions $\mathbb{F}^S \rightarrow \mathbb{F}$:
- $f_{coeff(h^*)}$: the vector of coefficients of $h^*$
- $f_z$: The vector $z$

Then, the verifier can:
- Evaluate $g_z(r)$ by querying $f_z$ at the vectors
	- $q^{(1)} = (A_1(r), ..., A_S(r))$
	- $q^{(2)} = (B_1(r), ..., B_S(r))$
	- $q^{(3)} = (C_1(r), ..., C_S(r))$
- Evaluate $h^*(r)$ by querying $f_{coeff(h^*)}$ at $q^{(4)} = (1, r, r^2, ..., r^S)$

 **Linear Interactive Proof (LIP)** vs. linear PCP:
 - When querying the *same* polynomial using queries $q^{(1)}, ..., q^{(k)}$, a LIP ensures that all queries are answered using the *same* linear function
 - We can add query $q^{(k + 1)} = \sum_{i = 1}^k \beta_i q^{(i)}$ for randomly chosen $\beta_i$
 - Given answers $a_1, ..., a_{k + 1}$, the verifier accepts iff. $a_{k + 1} = \sum_{i = 1}^k \beta_i a_i$

=> We add $q^{(5)} = \sum_{i = 1}^3 \beta_i q^{(i)}$

**SRS:** For a randomly chosen $\alpha$ and each $i = 1, ..., 5$ and $j = 1, ..., S$, the SRS contains the pair $(g^{q_j^{(i)}}, g^{\alpha q_j^{(i)}})$. (Only depends on the circuit!)

**Verification key**: $(g, g^{\alpha}, g^{\mathbb{Z}_H(r)}, g^{\beta_1}, g^{\beta_2}, g^{\beta_3})$

**Proof**: Using the SRS and additive homomorphism, the prover computes:
- $(g_1, g'_1) := (g^{f_z(q^{(1)})}, g^{\alpha \cdot f_z(q^{(1)})})$
- $(g_2, g'_2) := (g^{f_z(q^{(2)})}, g^{\alpha \cdot f_z(q^{(2)})})$
- $(g_3, g'_3) := (g^{f_z(q^{(3)})}, g^{\alpha \cdot f_z(q^{(3)})})$
- $(g_4, g'_4) := (g^{f_{coeff(h^*)}(q^{(4)})}, g^{\alpha \cdot f_{coeff(h^*)}(q^{(4)})})$
- $(g_5, g'_5) := (g^{f_z(q^{(5)})}, g^{\alpha \cdot f_z(q^{(5)})})$

**Verification**:
- $e(g_1, g_2) = e(g_3, g) \cdot e(g^{\mathbb{Z}_H(r)}, g_4)$
	- Verifies that $g(t)$ vanishes on $H$!
- $\prod_{i = 1}^3 e(g^{\beta_i}, g_i) = e(g_5, g)$
	- The verification from the LIP above
- For $i = 1, ..., 5$:  $e(g_i, g^{\alpha}) = e(g, g'_i)$
	- Under the **Knowledge of Exponent Assumption (KEA)**, this ensures that the prover actually knows the exponents of the sent values

*Question: How is it possible that there is nothing circuit-specific in the verification algorithm & verification key?!*
- $g^{\mathbb{Z}_H(r)}$ is used in the verification, so it's specific to the setup
- The prover could not have used different values for $q^{(1)}$,  $q^{(2)}$, $q^{(3)}$, and $q^{(5)}$ (corresponding to a different circuit), because that would require knowing $r$

**Public inputs:** This is a subset of $z$. The prover can simply commit to the other parts of $z$, while the verifier computes the contribution of the public input on its own. For this, $g^{q_j^{i}}$ needs to be part of the verification key (but **not** $g^{\alpha q_j^{i}}$; this was the bug Ariel Gabizon found in the BCTV14 paper and ZCash!)

**Achieving Zero Knowledge** (sketch): The prover can pick random values $r_A, r_B, r_C \in \mathbb{F}$ and apply the protocol to:
$$
g'_z(t) := (A(t) + r_A\mathbb{Z}_H(t)) \cdot (B(t) + r_B\mathbb{Z}_H(t)) - (C(t) + r_C\mathbb{Z}_H(t))
$$
The rest of the protocol can be modified accordingly.

**Groth16**: Very influential variant of this SNARK which reduces the size of the proof from 10 to 3 group elements by:
- Introducing a LIP that makes 3 queries instead of 5
- Relying on the Algebraic Group Model instead of KEA, so all the $g'_i$ elements aren't needed anymore
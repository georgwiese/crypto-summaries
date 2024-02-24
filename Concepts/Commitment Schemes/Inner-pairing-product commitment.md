*Source: [[Proofs, Arguments, and Zero-Knowledge]], section 15.4.1*

Also called **AFGHO-commitment**.

Let $\mathbb{G}_1, \mathbb{G}_2, \mathbb{G}_T$ be a pairing-friendly triple of groups of order $p$. We'll express them as additive groups.
For a message $w \in \mathbb{G}_1^n$ and fixed vector $\mathbf{g} \in \mathbb{G}_2^n$, define:
$$
IPPCom(w) = \sum_{i = 1}^n e(w_i, g_i)
$$
This is similar to a [[Pedersen Commitment]], except that instead of using the message as a factor in a *scalar multiplication*, it is now a group element which is "multiplied" via the pairing.

**Rendering the commitment perfectly hiding**: Add $e(r, g)$ for a random $e \in \mathbb{G}_1$

**Computational binding**: Implied by the Decisional Diffie-Hellman assumption (DDH)
- -> Can only hold if $\mathbb{G}_1 \ne \mathbb{G}_2$ and there is no efficiently computable mapping $\phi: \mathbb{G}_1 \rightarrow \mathbb{G}_2$ such that $\phi(a + b) = \phi(a) + \phi(b)$
- -> Believed to be true for BLS12-381
- The assumption that DDH holds in both groups is called the *symmetric external Diffie-Hellman assumption (SXDH)*

**Commitment to field elements**: To compute to a vector of *field elements*, we can first compute an (unblinded) [[Pedersen Commitment]] using the *same* generator $h \in \mathbb{G}_1$ for each element:
$$
IPPCom(v) = \sum_{i = 1}^n e(v_i \cdot h, g_i) = e(h, \sum_{i = 1}^n v_i \cdot g_i)
$$
(note that the second way to compute it is far more efficient)
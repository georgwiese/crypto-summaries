[ePrint 2025/946](https://eprint.iacr.org/2025/946) | Lev Soukhanov, 2025

An indexed lookup argument where the looked-up values are *virtual* (not committed). Especially efficient for small tables ($m \ll n$).

## Setting

Given commitments to:
- An index array $I$ (size $n$)
- A table $T$ (size $m$)

Goal: evaluate the *pullback* $(I^*T)[i] := T[I[i]]$ at a random point $r$, without committing to $I^*T$.

## Pushforward

The dual of the pullback. Given $I: \{0, \ldots, n-1\} \to \{0, \ldots, m-1\}$ and $A \in \mathbb{F}^n$:

$$I_*A[j] = \sum_{i \mid I[i] = j} A[i]$$

**Duality lemma**: $\langle I_*A, B \rangle = \langle A, I^*B \rangle$.

## Protocol

Trade the pullback for a pushforward using the duality lemma:

$$I^*T(r) = \langle I^*T, \mathsf{eq}_r \rangle = \langle T, I_*\mathsf{eq}_r \rangle$$

where $\mathsf{eq}_r \in \mathbb{F}_{\text{ext}}^n$ is the vector of Lagrange basis evaluations at $r$.

**Steps:**
1. Prover commits to $I_*\mathsf{eq}_r \in \mathbb{F}_{\text{ext}}^m$.
2. Run [[Multivariate Sum-Check Protocol|sum-check]] for $\langle T, I_*\mathsf{eq}_r \rangle = e$, obtaining evaluation claims for $\widetilde{T}$ and $\widetilde{I_*\mathsf{eq}_r}$.
3. Open $T$ and $I_*\mathsf{eq}_r$ via polynomial commitments.

**Well-formedness** of $I_*\mathsf{eq}_r$ is proven via [[GKR]], using a [[LogUp & cq|LogUp]]-style fractional sum:

$$\sum_{0 \le i < n} \frac{\mathsf{eq}_r[i]}{c - I[i]} = \sum_{0 \le j < m} \frac{I_*\mathsf{eq}_r[j]}{c - j}$$

The verifier can evaluate $\widetilde{\mathsf{eq}}_r$ by itself, so no commitment to it is needed.

## Cost comparison vs. standard LogUp

Compared to [[LogUp & cq|LogUp + GKR]] with the standard indexed-to-unindexed reduction:
- **Saves**: commitment to $I^*T$ ($n$ base field elements, multiplied by $t$ for tuple lookups of size $t$) and multiplicities ($m$ elements)
- **Adds**: commitment to $I_*\mathsf{eq}_r$ ($m$ extension field elements) + extra sum-checks

Best when $m \ll n$ and table values are large (extension field elements or tuples). Also avoids the numerator overflow issue that affects standard LogUp accumulators. For example, it can be useful in [[Spark]].
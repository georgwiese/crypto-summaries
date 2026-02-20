[Georg Wiese (Powdr Labs), 2025](https://www.powdr.org/papers/twist_shout_logup_star.pdf)

Adapts [[Twist and Shout]] to work with hash-based commitment schemes, using the pushforward machinery from [[Logup*|logup*]].

**Motivation**: Twist and Shout rely on efficient commitments to *sparse* polynomials, which is natural for elliptic-curve-based schemes but expensive for hash-based schemes. Logup* provides a way around this.
## One-hot matrix commitment scheme

[[Logup*|Logup*]] implicitly contains a commitment scheme for matrices with one-hot rows $M \in \mathbb{F}^{n \times m}$. Instead of committing to the full matrix, commit to its dense encoding $M_{\text{dense}} \in \mathbb{F}^n$ (the column index of the 1 in each row).

To evaluate $\widetilde{M}(r_{\text{row}}, r_{\text{col}})$:
1. The prover commits to $P := {M_{\text{dense}}}_* \mathsf{eq}_{r_{\text{row}}} \in \mathbb{F}_{\text{ext}}^m$.
2. Prover proves well-formedness via [[GKR]], as in [[Logup*]].
3. The key identity is $\widetilde{M}(r_{\text{row}}, r_{\text{col}}) = \widetilde{P}(r_{\text{col}})$.

Derivation of (3):
$$
\begin{align}
\widetilde{P}(r_{\text{col}}) &= \langle P, \mathsf{eq}_{r_{\text{col}}} \rangle \\
&= \sum_{j=0}^{m-1} P[j] \cdot \widetilde{\mathsf{eq}}(\texttt{bits}(j), r_{\text{col}}) \\
&= \sum_{j=0}^{m-1} \left(\sum_{i \mid M_{\text{dense}}[i] = j} \widetilde{\mathsf{eq}}(\texttt{bits}(i), r_{\text{row}})\right) \cdot \widetilde{\mathsf{eq}}(\texttt{bits}(j), r_{\text{col}}) \\
&= \sum_{(i,j) \mid M[i,j] = 1} \widetilde{\mathsf{eq}}(\texttt{bits}(i) \| \texttt{bits}(j),\; r_{\text{row}} \| r_{\text{col}}) \\
&= \widetilde{M}(r_{\text{row}}, r_{\text{col}})
\end{align}
$$

The third step expands the pushforward definition, the fourth uses the product structure of $\widetilde{\mathsf{eq}}$, and the last step is the definition of the multilinear extension of a matrix with one-hot rows (only $n$ nonzero terms).
### Batching

Naively, opening $d$ one-hot matrices requires $d$ pushforward commitments. Batching reduces this to one. Given $d$ dense encodings $M^{(1)}_{\text{dense}}, \ldots, M^{(d)}_{\text{dense}} \in \mathbb{F}^n$ of one-hot matrices $M^{(i)} \in \{0,1\}^{n \times m}$:
1. Concatenate: $M^{(*)}_{\text{dense}} := M^{(1)}_{\text{dense}} \| \cdots \| M^{(d)}_{\text{dense}} \in \mathbb{F}^{n \cdot d}$.
	- This does not require new commitments: $\widetilde{M_{\text{dense}}^{(*)}}(x) = \sum_{i = 1}^d \widetilde{eq}(\texttt{bits}(i), x) \cdot \widetilde{M_{\text{dense}}^{(i)}}$.
2. Use $\widetilde{M}^{(i)}(x_{\text{row}}, x_{\text{col}}) = \widetilde{M}^{(*)}(x_{\text{row}}, \texttt{bits}(i), x_{\text{col}})$ to translate $d$ claims to $\widetilde{M}^{(i)}$ to claims about $\widetilde{M}^{(*)}$.
3. Reduce $d$ claims to one via [[Reducing Multiple Evaluations to One]].
4. Open via a single pushforward $P := {M^{(*)}_{\text{dense}}}_* \mathsf{eq}_{r_{\text{row}}} \in \mathbb{F}_{\text{ext}}^m$.
## Twist and Shout via logup*
### Shout via logup*

Directly applies the one-hot commitment scheme with batching. With decomposition parameter $d$ (as in [[Twist and Shout#Parameter $d$: splitting the one-hot encoding|original Shout]]):
- Given $d$ dense address vectors $\mathsf{ra}^{(i)}_{\text{dense}} \in \mathbb{F}^T$
- Given memory content $\mathsf{Val} \in \mathbb{F}^K$
- Commit to **one** batched pushforward $P := {\mathsf{ra}^{(*)}_{\text{dense}}}_* \mathsf{eq}_{r_{\text{row}}} \in \mathbb{F}_{\text{ext}}^{\sqrt[d]{K}}$

For $d = 1$, this is equivalent to using [[Logup*|logup*]] directly. For large memories, $d > 1$ reduces the pushforward cost from $K$ to $\sqrt[d]{K}$ extension field elements.
### Twist via logup*

Same idea applied to the read/write case:
- Given $d$ read address vectors $\mathsf{ra}^{(i)}_{\text{dense}} \in \mathbb{F}^T$
- Given $d$ write address vectors $\mathsf{wa}^{(i)}_{\text{dense}} \in \mathbb{F}^T$
- Given Increment vector $\mathsf{Inc}_{\text{val}} \in \mathbb{F}^T$ (dense; position determined by write address)
- One batched pushforward $P \in \mathbb{F}_{\text{ext}}^{\sqrt[d]{K}}$ (shared across all $2d$ address matrices)
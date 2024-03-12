by Blum at all (1995)

Sources:
- [Explanation (YouTube)](https://youtu.be/dmVweFbJsxw?si=26si_BTIIcVzPb0m&t=4474)
- [[Unlocking the lookup singularity with Lasso]]

### Main Idea
The problem they tried to solve is to force an untrusted memory to prove it worked correctly, with very low space requirements of the verifier.

The verifier maintains two sets, which can be compressed and updated incrementally via permutation-invariant fingerprinting (see [[Permutation Check via Product Check]]):
- **Read Set**: The tuples $(address_i, value_i, count_i)$ returned by the prover
- **Write Set**: $(address_i, value_i, count_i)$ written to memory

Then, the verifier behaves as follows:
- At the beginning of the execution, (s)he writes for each address $a_i$ the tuple $(a_i, 0, 0)$ (or some other initial value)
- Each read returning $(a_i, v_i, c_i)$ is followed by a write of $(a_i, v_i, c_i + 1)$
- At the end of the execution, the verifier reads all addresses

If and only if the prover is honest, the read and write *multi*-sets will be equal.

### Application: Indexed Lookup
Source: [[Unlocking the lookup singularity with Lasso]]

Suppose we have:
- A multi-linear extension $\widetilde{t}: \mathbb{F}^{\log M} \rightarrow \mathbb{F}$  of some table (i.e., read-only memory)
- A multi-linear extension $\widetilde{i}: \mathbb{F}^{\log m} \rightarrow \mathbb{F}$ of indices
- A multi-linear extension $\widetilde{v}: \mathbb{F}^{\log m} \rightarrow \mathbb{F}$ of values

We want to show that:
$$
\forall{j \in \{0, 1\}}^{\log m}: \widetilde{v}(j) = \widetilde{t}(\mathtt{bits}(\widetilde{i}(j)))
$$
where $\mathtt{bits}: \mathbb{F} \rightarrow \mathbb{F}^{\log N}$  is the function that maps a field element to its bit representation.

We can show this is the case by simulating the offline memory verifier in a SNARK.

Specifically, the prover commits to two polynomials, $\widetilde{c}_{read}: \mathbb{F}^{\log m} \rightarrow \mathbb{F}$ and $\widetilde{c}_{final}: \mathbb{F}^{\log M} \rightarrow \mathbb{F}$, containing the counts returned by the offline memory prover for the "normal" and final read operations.

The memory operations were performed correctly if $RS = WS$ where:
$$
\begin{equation}
\begin{split}
RS = &\{(\widetilde{i}(j), \widetilde{v}(j), \widetilde{c}_{read}(j)): j \in \{0, 1\}^{\log m}\} \\ &\cup \{(\mathtt{to\_field}(j), \widetilde{t}(j), \widetilde{c}_{final}(j)): j \in \{0, 1\}^{\log M}\}
\end{split}
\end{equation}
$$
$$
\begin{equation}
\begin{split}
WS = &\{(\mathtt{to\_field}(j), \widetilde{t}(j), 0): j \in \{0, 1\}^{\log M}\} \\ &\cup \{(\widetilde{i}(j), \widetilde{v}(j), \widetilde{c}_{read}(j) + 1): j \in \{0, 1\}^{\log m}\}
\end{split}
\end{equation}
$$
where $\mathtt{to\_field}$ is also a multi-linear polynomial:
$$
\mathtt{to\_field}(x_1, ..., x_{\log M}) := \sum_{i = 1}^{\log M} 2^{i - 1} \cdot x_i
$$

The two multi-sets can be shown to be equal using the [[Permutation Check via Product Check]]! In particular, the grand product can be computed using a layered arithmetic circuit of depth $O(\log m + \log M)$ and proven using [[GKR]].
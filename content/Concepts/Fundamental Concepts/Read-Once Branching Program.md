A **read-once branching program (ROBP)** of width $w$ over alphabet $\Sigma = \{0,1\}^b$ is a layered directed graph with $n+1$ layers, where:
- Layer 0 has a single source vertex
- Each layer has at most $w$ vertices (representing $\log_2(w)$ bits of memory)
- Each non-sink vertex has $2^b$ outgoing edges (one per symbol)
- Sinks are labeled with field elements

**Evaluation**: Follow edges labeled $x_1, x_2, \ldots, x_n$ from source to sink.

## Computing the MLE ([Holmgren-Rothblum 2018](https://eprint.iacr.org/2018/861))

Process the branching program **backwards** (from sinks to source), computing the MLE of "the function starting from vertex $v$" for each vertex.

1. **Base case**: For each sink vertex $v$ with label $\alpha$, set $\hat{f}_v = \alpha$

2. **Inductive step**: For vertex $v$ in layer $i$:
$$
\hat{f}_v(\zeta, z) = \sum_{\sigma \in \{0,1\}^b} \widetilde{eq}(\zeta, \sigma) \cdot \hat{f}_{\Gamma(v,\sigma)}(z)
$$
   where $\Gamma(v, \sigma)$ is the vertex reached by following edge $\sigma$ from $v$.

3. **Result**: $\hat{f}(z) = \hat{f}_{\text{source}}(z)$

For Boolean $\zeta$, the $\widetilde{eq}$ selects exactly one term (the edge actually taken). For field elements, it smoothly interpolates.

**Cost**: $O(n \cdot w^2 \cdot 2^b)$ — per layer: $2^b$ to compute $\widetilde{eq}(z_i, \cdot)$, then $w^2$ to combine values across vertices.

Used in [[Jagged Polynomial Commitments]].

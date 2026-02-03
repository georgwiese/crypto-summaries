*Source: [Jagged Polynomial Commitments](https://eprint.iacr.org/2025/917) — Hemo, Jue, Rabinovich, Roh, Rothblum (Succinct, 2025)*

Sparse PCS for zkVM multi-table traces. Commits to entire trace as single dense polynomial while emulating access to individual columns.

## Jagged Functions

A **jagged function** $p: \{0,1\}^n \times \{0,1\}^k \to \mathbb{F}$ has heights $(h_y)_{y \in \{0,1\}^k}$ where $p(x,y) = 0$ for $x \geq h_y$. Total non-zero size $M = \sum_y h_y = 2^m$.

**Dense representation**: Pack non-zeros into $q: \{0,1\}^m \to \mathbb{F}$ using cumulative heights $t_y = \sum_{y' \leq y} h_{y'}$:
- $\text{col}_t(i) = \min\{y : i < t_y\}$
- $\text{row}_t(i) = i - t_{y-1}$

## Protocol

**Commitment**: Commit to dense $\hat{q}$, send heights $(t_y)$ in clear.

**Evaluation** of $\hat{p}(z_r, z_c) = v$: Run [[Multivariate Sum-Check Protocol]] on
$$v = \sum_{i \in \{0,1\}^m} \hat{q}(i) \cdot \hat{f}_t(z_r, z_c, i)$$
where $f_t(z_r, z_c, i) = 1$ iff $\text{row}_t(i) = z_r$ and $\text{col}_t(i) = z_c$.

Reduces to: one claim on $\hat{q}$ (to underlying PCS) + one claim on $\hat{f}_t$ (verifier computes directly).

## Computing $\hat{f}_t$

$$\hat{f}_t(z_r, z_c, i) = \sum_{y \in \{0,1\}^k} \widetilde{eq}(z_c, y) \cdot \hat{g}(z_r, i, t_{y-1}, t_y)$$

where $g(a, b, c, d) = 1$ iff $b = a + c \land b < d$.

### Branching Program for $g$

Width-4 [[Read-Once Branching Program]] with 2-bit state $(\text{carry}, \text{lt})$. Processes inputs LSB-first, reading $(a_i, b_i, c_i, d_i)$ at each step:
1. Reject if $\text{LSB}(a_i + c_i + \text{carry}) \neq b_i$
2. $\text{carry} \leftarrow \text{MSB}(a_i + c_i + \text{carry})$
3. $\text{lt} \leftarrow (d_i > b_i) \lor (d_i = b_i \land \text{lt})$

**Example**: $g(1, 2, 1, 5) = g(\texttt{0b001}, \texttt{0b010}, \texttt{0b001}, \texttt{0b101})$ — check $2 = 1 + 1$ and $2 < 5$

| $i$ | $a_i$ | $b_i$ | $c_i$ | $d_i$ | carry | lt |
|-----|-------|-------|-------|-------|-------|-----|
| 0   | 1     | 0     | 1     | 1     | 1     | 1 (d>b) |
| 1   | 0     | 1     | 0     | 0     | 0     | 0 (b>d) |
| 2   | 0     | 0     | 0     | 1     | 0     | 1 (d>b) |

Result: $g = 1$ ✓

For any computation that can be described using a low-width branching program, there is an efficient algorithm to compute its multilinear extension.

## Efficiency

- **Prover**: $5 \cdot 2^m$ multiplications
- **Verifier**: $O(m \cdot 2^k)$ (depends only on log-trace-area, not individual heights)

## Comparison to [[Spark]]

|                     | Spark            | Jagged   |
| ------------------- | ---------------- | -------- |
| Additional oracles  | 7+               | None     |
| Verifier depends on | Sparsity pattern | Only $m$ |

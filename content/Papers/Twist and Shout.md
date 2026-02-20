[ePrint 2025/105](https://eprint.iacr.org/2025/105) | Srinath Setty, Justin Thaler, 2025
[Jolt docs](https://jolt.a16zcrypto.com/how/twist-shout.html) | [a16z blog post](https://a16zcrypto.com/posts/article/introducing-twist-and-shout/) | [zkSummit 13 talk](https://www.youtube.com/watch?v=nEEFjyTK8OI)

Replacements for [[Lasso]] and Spice ([[Offline Memory Checking]]) in [[Jolt]], based purely on the [[Multivariate Sum-Check Protocol]]. No grand product arguments.
- **Shout**: Read-only memory (a.k.a. lookup arguments)
- **Twist**: Read/write memory

## Shout

### Lookups as a matrix-vector product

Let $\mathsf{rv} \in \mathbb{F}^T$ be the result of $T$ reads into a read-only memory $\mathsf{Val} \in \mathbb{F}^K$. At each cycle $j \in [T]$, encode the read address as a one-hot vector $\mathsf{ra}_j \in \{0,1\}^K$. Stack these into a matrix $\mathsf{ra} \in \{0,1\}^{T \times K}$, which is sparse. Then the read values are simply:
$$\mathsf{rv} = \mathsf{ra} \cdot \mathsf{Val}$$

To verify this via sum-check, pick random $r_{\text{cycle}} \in \mathbb{F}^{\log T}$ and check:
$$\widetilde{\mathsf{rv}}(r_{\text{cycle}}) = \sum_{k \in \{0,1\}^{\log K}} \widetilde{\mathsf{ra}}(k, r_{\text{cycle}}) \cdot \widetilde{\mathsf{Val}}(k)$$

The prover commits only to $\widetilde{\mathsf{ra}}$ (the one-hot addresses). The read-value polynomial $\widetilde{\mathsf{rv}}$ is **virtual**: it is fully determined by $\widetilde{\mathsf{ra}}$ and $\widetilde{\mathsf{Val}}$, so it need not be committed. $\widetilde{\mathsf{Val}}$ is either public, committed or evaluable by the verifier in $O(\log K)$ time for MLE-structured tables.

**Prover cost** ($d = 1$): The sum-check is over $K \cdot T$ terms, but only $T$ are non-zero (one per row), so the prover runs in $O(K + T)$ time. Commitment: $T$ non-zero values committed (all 1s), plus $O(K \log K)$ field multiplications.

### Committing to $\widetilde{\mathsf{ra}}$

They use an Elliptic Curve based commitment scheme, so the commitment cost for $\widetilde{\mathsf{ra}}$ is only proportional to $T$ (just one group operation per bit). On top of this, the one-hot property has to be proven, by proving booleanity of all the entries and proving that the sum along the memory dimension is 1.

See [[Twist and Shout via logup*]] for alternative ways to commit to $\widetilde{\mathsf{ra}}$, applicable to hash-based commitments.
### Parameter $d$: splitting the one-hot encoding

**Motivation**: The commitment key has $K \cdot T$ entries, which is a problem for large memories. Also, the evaluation proofs become more expensive, but I skipped the details of that.

Fix $d \geq 1$ and let $N = K^{1/d}$. Decompose each one-hot vector $e_z \in \{0,1\}^K$ as a tensor product of $d$ smaller one-hot vectors:
$$e_z = v_1 \otimes v_2 \otimes \cdots \otimes v_d, \quad v_i \in \{0,1\}^N$$

The prover commits to $d$ polynomials $\widetilde{\mathsf{ra}}_1, \ldots, \widetilde{\mathsf{ra}}_d$ (each over $\mathbb{F}^{\log N + \log T}$) instead of one polynomial over $\mathbb{F}^{\log K + \log T}$. The sum-check becomes:
$$\widetilde{\mathsf{rv}}(r') = \sum_{\substack{k = (k_1, \ldots, k_d) \in \{0,1\}^{d \log N} \\ j \in \{0,1\}^{\log T}}} \widetilde{\mathsf{eq}}(r', j) \cdot \left(\prod_{\ell=1}^{d} \widetilde{\mathsf{ra}}_\ell(k_\ell, j)\right) \cdot \widetilde{\mathsf{Val}}(k)$$

**Tradeoff**: The $d$ polynomials share one SRS, so the commitment key drops from $K$ to $K^{1/d}$ group elements. The number of commitments per cycle grows from 1 to $d$ ($d \cdot T$ non-zero values total, each a 1). The degree in each variable increases from 3 to $2 + d$, raising prover field work.

## Twist

Read/write memory with $K$ cells and $T$ cycles (alternating reads and writes). The data:

|                                            | Description                          | Committed? |
| ------------------------------------------ | ------------------------------------ | ---------- |
| $\mathsf{ra} \in \{0,1\}^{T \times K}$     | One-hot read addresses (as in Shout) | yes        |
| $\mathsf{wa} \in \{0,1\}^{T \times K}$     | One-hot write addresses              | yes        |
| $\mathsf{Inc} \in \mathbb{F}^T$            | Increment per cycle (see below)      | yes        |
| $\mathsf{rv} \in \mathbb{F}^T$             | Read values                          | virtual    |
| $\mathsf{wv} \in \mathbb{F}^T$             | Write values                         | virtual    |
| $\mathsf{Val} \in \mathbb{F}^{K \times T}$ | Value of each cell at each cycle     | virtual    |

Unlike Shout, $\mathsf{Val}$ changes over time, so $\mathsf{rv}(j) = \sum_k \mathsf{ra}(k, j) \cdot \mathsf{Val}(k, j)$ is no longer a simple matrix-vector product with a static table.

### Increments

Since only one cell $k^*$ is written per cycle $j$, define:
$$\mathsf{Inc}(j) := \mathsf{wv}(j) - \mathsf{Val}(k^*, j)$$
*Note: this is one-dimensional, unlike in the paper. See the [Jolt book](https://jolt.a16zcrypto.com/how/twist-shout.html#wv-virtualization).*

Given commitments to $\widetilde{\mathsf{ra}}$, $\widetilde{\mathsf{wa}}$, and $\widetilde{\mathsf{Inc}}$, evaluations of the virtual polynomials can be obtained via three sum-checks.
### Sum-checks

**Val-evaluation sum-check**: $\mathsf{Val}(k, j)$ is the sum of all prior increments to cell $k$:
$$
\widetilde{\mathsf{Val}}(r_{\text{addr}}, r_{\text{cycle}}) = \sum_{j' \in \{0,1\}^{\log T}} \widetilde{\mathsf{Inc}}(j') \cdot \widetilde{\mathsf{wa}}(r_{\text{addr}}, j') \cdot \widetilde{\mathsf{LT}}(j', r_{\text{cycle}})
$$
where $\mathsf{LT}(j', j) = 1$ iff $\mathsf{int}(j') < \mathsf{int}(j)$. At the end, the verifier needs evaluations of $\widetilde{\mathsf{Inc}}$ and $\widetilde{\mathsf{wa}}$ (from commitments) and $\widetilde{\mathsf{LT}}$ (computable in $O(\log T)$ time).

**Read-checking sum-check**:  Like Shout, but summing over both $k$ and $j$ since $\mathsf{Val}$ is time-varying:
$$\widetilde{\mathsf{rv}}(r') = \sum_{(k, j) \in \{0,1\}^{\log K + \log T}} \widetilde{\mathsf{eq}}(r', j) \cdot \widetilde{\mathsf{ra}}(k, j) \cdot \widetilde{\mathsf{Val}}(k, j)$$
This requires evaluating $\widetilde{\mathsf{Val}}$ at a random point, which is handled by the Val-evaluation sum-check above.

**Write-checking sum-check**: The write value is the old cell value plus the increment:
$$\widetilde{\mathsf{wv}}(r') = \sum_{(k, j) \in \{0,1\}^{\log K + \log T}} \widetilde{\mathsf{eq}}(r', j) \cdot \widetilde{\mathsf{wa}}(k, j) \cdot \left(\widetilde{\mathsf{Val}}(k, j) + \widetilde{\mathsf{Inc}}(j)\right)$$
Also requires a $\widetilde{\mathsf{Val}}$ evaluation, obtained from the same Val-evaluation sum-check (run once, shared by both).

**Parameter $d$**: Just like Shout, the sparse matrices can be committed as $d$ separate matrices.

**Locality benefit**: A read/write to a cell last accessed $2^i$ cycles ago costs only $O(i)$ field multiplications, because the relevant vectors become sparser in early sum-check rounds.

## Recovering dense addresses

The zkVM naturally represents addresses as field elements: $\mathsf{raf} \in \mathbb{F}^T$ where $\mathsf{raf}(j)$ is the address read at cycle $j$ (and similarly $\mathsf{waf}$). These are virtual polynomials, recoverable from the one-hot commitments via:
$$\widetilde{\mathsf{raf}}(r_{\text{cycle}}) = \sum_{k \in \{0,1\}^{\log K}} \widetilde{\mathsf{to\_field}}(k) \cdot \widetilde{\mathsf{ra}}(k, r_{\text{cycle}})$$
where $\mathsf{to\_field}(k_1, \ldots, k_{\log K}) = \sum_i 2^{i-1} k_i$ converts binary to a field element. The verifier runs a sum-check over $k$; at the end it needs $\widetilde{\mathsf{ra}}$ at a random point (from commitment) and $\widetilde{\mathsf{to\_field}}$ at a random point (computable in $O(\log K)$ time).
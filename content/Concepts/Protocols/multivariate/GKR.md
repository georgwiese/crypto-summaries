*Source: [[Proofs, Arguments, and Zero-Knowledge]], Section 4.6.*

An IP for general-purpose computation due to Goldwasser, Kalai, and Rothblum (2008).
### The Setting
We assume the computation in a *layered arithmetic circuit* of low depth (as verification time will be linear in the depth of the circuit).

Notation:
- Layers are numbered from $0$ (*output* layer) to $d$
- At each layer, there are $S_i = 2^{k_i}$ gates
- $W: \{0, 1\}^{k_i} \rightarrow \mathbb{F}$ denotes the functions that maps binary gate label in layer $i$ to its output
- *Wiring predicates* $add_i$ and $mult_i$ are functions $\{0, 1\}^{k_i + 2 k_{i + 1}} \rightarrow \{0, 1\}$ that map one gate label of layer $i$ and two of layer $i + 1$ to a $1$ iff. they correspond to the two inputs and one output of an addition or multiplication, respectively
### The protocol
$\widetilde{W}$ can be explicitly expressed as follows:
$$
\widetilde{W}_i(z) = \sum_{b,c \in \{0,1\}^{k_i+1}} \widetilde{add}_i(z, b, c) \left( \widetilde{W}_{i+1}(b) + \widetilde{W}_{i+1}(c) \right) + \widetilde{mult}_i(z, b, c) \left( \widetilde{W}_{i+1}(b) \cdot \widetilde{W}_{i+1}(c) \right)

$$
This follows from the fact that multilinear extensions are unique and that the right-hand side is a multilinear polynomial.

Now, the algorithm is as follows:
- The prover provides the claimed output, from which the verifier can reconstruct the claimed polynomial $\widetilde{W_0}$
- The verifier can test for equality at a random point by applying the [[Multivariate Sum-Check Protocol]] to the expression above
- At the end of the sum-check protocol, the verifier needs to evaluate $\widetilde{add_i}$, $\widetilde{mult_i}$, and $\widetilde{W_{i+1}}$ at a random point. For the wiring predicates, we assume that this can be done efficiently.
- For the evaluation of $\widetilde{W_{i+1}}$ at two random points, we apply a reduction to one point -> [[Reducing Multiple Evaluations to One]]
- In the final layer, $\widetilde{W_{d}}$ is simply the multilinear extension provided by the prover (i.e., all inputs are public)

In more detail:
![[gkr.png]]
### Runtimes & Soundness errors
**Communication cost**: $O(S_0 + d \log S)$
**Verifier cost**: $n + d \log S + t + S_0$, where $t$ is the time needed for all evaluations of the wiring predicates
- $t$ in $O(\log S)$ for many common wiring patterns, including matrix multiplication, Fast Fourier Transforms, distinct elements, ...
- However, there is no clean categorization of when this is the case
- If it's not possible, one can resort to *polynomial commitment schemes*, resulting in an *argument* instead of an interactive proof
**Prover cost**: Analogous to the cost analysis in [[Super-Efficient IP for Matrix Multiplication]], we can exploit the fact that wiring predicates are sparse
**Soundness error**: $O(d \log(S) / |\mathbb{F}|)$
- If the claimed output does not equal the actual output, in at least one round, the prover must have sent a polynomial that is different from the one the honest prover would have sent
- The probability of this being undetected is $O(\log(S) / |\mathbb{F}|)$, by the [[Schwartz-Zippel Lemma]]
*Sources:
- [[Proofs, Arguments, and Zero-Knowledge]], chapter 8
- [generalized CCS backend / DSL proposal](https://hackmd.io/@levs57/By2yBLefJe): Nice blog post by Lev Soukhanov
### Warm-up: Clover (BTVW14)
Achieves a verification time independent of the circuit depth!

**Idea**: Instead of committing to the witness, commit to the full *transcript*, i.e., the evaluation of every single gate in the circuit
-> Higher commitment cost than [[GKR]]

- The prover claims to "hold" an extension $\widetilde{W}$ of a correct transcript.
- For a circuit of $2^k$ gates, let's define $add, mult, io: \{0, 1\}^{3k} \rightarrow \{0, 1\}$ which takes 3 gate labels $(a, b, c)$ as inputs and output $1$ iff.:
	- $add, mult$: $a$ has $b$ and $c$ as inputs and is an addition / multiplication gate
	- $io$: gate $a$ is either a gate from the circuit input $x$ and $b = c = \mathbf{0}$ or one of the output gates, and gates $b$ and $c$ are the inputs to $a$
- Also, $I_{x, y}: \{0, 1\}^k \rightarrow \{0, 1\}$ with $I_{x, y}(a) = x_a$ if $a$ is an input gate, $I_{x, y}(a) = y_a$ if $a$ is an output gate, and  $I_{x, y}(a) = 0$ otherwise
- Then, $W$ is a correct transcript iff. the following function vanishes on the boolean hypercube: $G_{x,y,W}(a, b, c) = io(a, b, c) \cdot (I_{x}(a) - W(a)) + \text{add}(a, b, c) \cdot (W(a) - (W(b) + W(c))) + \text{mult}(a, b, c) \cdot (W(a) - W(b) \cdot W(c))$
--> Apply the [[Multivariate Zero Check Protocol]]
- To evaluate $\widetilde{W}$ at 3 random inputs, the verifier can reduce to one point (see [[Reducing Multiple Evaluations to One]]) and then query the second prover (i.e., request an evaluation prove from the PCS)
--> Possible to implement prover in $O(S)$ time

### Spartan (Setty, 20)
An R1CS instance is specified by 3 matrices $A$, $B$, and $C$ and is *satisfiable* if there is a vector $z$ with $z_1 = 1$ such that:
$$
(A \cdot z) \circ (B \cdot z) = C \cdot z
$$
This can be proven by showing that the following polynomial vanishes on the boolean hypercube:
$$
g_Z(X) := \left( \sum_{b \in \{0,1\}^{\log_2 n}} \widetilde{A}(X, b) \cdot \widetilde{z}(b) \right) \cdot \left( \sum_{b \in \{0,1\}^{\log_2 n}} \widetilde{B}(X, b) \cdot \widetilde{z}(b) \right) - \left( \sum_{b \in \{0,1\}^{\log_2 n}} \widetilde{C}(X, b) \cdot \widetilde{z}(b) \right) = 0
$$
This can be shown using the [[Multivariate Zero Check Protocol]].

To evaluate the $g_Z(X)$ at a random point, the [[Multivariate Sum-Check Protocol]] can be executed 3 times in parallel using the same randomness. The prover can be implemented in time *linear* time in the number of non-zero entries in $A$, $B$, and $C$ using [[Spark]].
## Generalization to CCS
I think this is best described in [Lev's post](https://hackmd.io/@levs57/By2yBLefJe):
- You can see R1CS of a very simple AIR of *virtual columns* $a = Az$, $b = Bz$, and $c = Cz$: `a * b = c`. Note that this already allows to express copy constraints, because $A, B, C$ can be such that the same element in $z$ is selected multiple times.
- CCS generalizes this notion and allows for number of virtual columns (with a matrix each) and any number of constraints between them.
- As a result, CCS generalizes R1CS, PlonK and AIR.
- Spartan generalizes to CCS.
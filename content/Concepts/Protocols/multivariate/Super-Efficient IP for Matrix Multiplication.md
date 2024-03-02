by #JustinThaler 
A protocol for evaluating $\tilde{f_C}$ at any given point $(r_1, r_2) \in \mathbb{F}^{\log n \times \log n}$ where $C = AB$ for two matrices $A$ and $B$. In contrast to Freivald's algorithm, it does not need to send the result matrix $C$, which requires $O(n^2)$ communication.

Key insight: $\tilde{f_C}(x, y) = \sum_{b \in \{0, 1\}^{\log n}} \tilde{f_A}(x, b) \cdot \tilde{f_B}(b, y)$
- If $\tilde{f_A}$ and $\tilde{f_B}$ is multilinear, so is $\tilde{f_C}$
- The multilinear extension of $C$ is unique
-> Apply sum-check protocol to $g(z) := \tilde{f_A}(r_1, z) \cdot \tilde{f_B}(z, r_2)$
### Runtime Analysis
**Verifier runtime**: $O(n^2)$
**Prover runtime**:
- To specify each polynomial $g_k(X_k)$, $P$ can send $g_k(0)$, $g_k(1)$, and $g_k(2)$
- 3 implementation with increased efficiency:
	- Method 1: By caching terms of the Lagrange basis polynomials, $g$ can be evaluated in $O(n^2)$ time. It needs to be evaluated at $3 \cdot n / 2^k$ points in round $k$, which leads to an $O(n^3)$ runtime
	- Method 2:
		- In each round $k$, each term in the sum has the structure:
		  $g(r_1, ..., r_{k-1}, {0, 1, 2}, b_{k + 1}, ..., b_{\log n})$ with $(b_{k + 1}, ..., b_{\log n}) \in \{0, 1\}^{\log n - k}$
		- Because the value $b_i$ are binary, each entry of the input matrices only contributes to 3 such terms
		- -> Can compute all terms with one pass over the matrices ($O(n^2)$)
		- -> Total runtime $O(\log n \cdot n^2)$
	- Method 3: Shares work across rounds -> $O(n^2)$ 
### Applications
- Super-efficient IP for counting triangles:
	- Apply the sum-check protocol to $\sum_{x, y \in {1, ..., n}} \tilde{f_{A^2}}(x, y) \cdot \tilde{f_{A}}(x, y)$
	- At the end of the protocol, the verifier needs to compute $\widetilde{f}_{A^2}(r_1, r_2)$ and $\widetilde{f}_{A}(r_1, r_2)$ -> [[Reducing Multiple Evaluations to One]]
	- $\tilde{f_{A}}(r_1, r_2)$ can be computed unaided, for $\tilde{f_{A^2}}(r_1, r_2)$, it can invoke the matrix multiplication protocol
- Super-efficient IP for matrix powers:
	- Goal compute $A^k$, where $k$ is a power of $2$
	- -> $\log k$ applications of the matrix multiplication IP, reducing multiple evaluations to 1 after each step
- In General, can be used in Combination with [[GKR]] as a subroutine, e.g. do prove graph diameters
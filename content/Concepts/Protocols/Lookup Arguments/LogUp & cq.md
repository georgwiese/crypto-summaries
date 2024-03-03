Sources:
- [[Multivariate lookups based on logarithmic derivatives]]
- [[Improving logarithmic derivative lookups using GKR]]
- [Understanding LogUp: A Royal Road](https://building-babylon.net/2024/02/14/a-royal-road-to-logup/) (Ben's Blog post)
- [[cq - Cached quotients for fast lookups]]

### Background: Lookup argument via product check
Analogously to the the [[Permutation Check via Product Check]], for multi-set $A$ and set $B$, we have $A \subseteq B$ if there exist values $\{m_b | b \in B\}$ such that the following two polynomials are equal:
$$
p_A(X) := \prod_{a \in A} (X - a)
$$
$$
p_B(X) := \prod_{b \in B} (X - b)^{m_b}
$$
The values $\{m_b | b \in B\}$ are the *multiplicities*, i.e., $m_b$ denotes the number of times $b$ appears in $A$.

The problem with this approach is that computing the exponentiation requires a bit-decomposition of $m_b$, which adds logarithmic overhead.
### Logarithmic Derivative
Applying the logarithm to and the taking the derivative of $p_A$ and $p_B$, we can transform the product check into a sum check over fractions:
$$
\sum_{a \in A} \frac{1}{X - a} \stackrel{?}{=} \sum_{b \in B} \frac{m_b}{X - b}
$$
### LogUp based on fractional sum check
([[Multivariate lookups based on logarithmic derivatives]])

The paper describes the protocol in the multivariate setting, but this can be adopted to the univariate setting by using the [[Univariate Sum-Check Protocol]] and [[Univariate Zero-Check Protocol]] instead of the multivariate variants.

#### Example: Looking up one column, no batching
Let $f_A, f_B, m: \mathbb{F}^n \rightarrow \mathbb{F}$ be the two multilinear polynomials that map each element of the boolean hypercube $H$ to an element in $A$, $B$, and the multiplicity respectively.

To show that the equality above holds, the verifier first samples a random $x \in \mathbb{F}$. Then, the prover needs to show that:
$$
\sum_{\vec{x} \in H} \frac{1}{x - f_A(\vec{x})} - \frac{m(\vec{x})}{x - f_B(\vec{x})}
$$
The problem is that the expression we're summing over is *not* a polynomial, so the sum-check protocol is not directly applicable!

To get around this, prover interpolates and commits to a polynomial $h$ such that:
$$
\forall \vec{x} \in H: h(\vec{x}) = \frac{1}{x - f_A(\vec{x})} - \frac{m(\vec{x})}{x - f_B(\vec{x})}
$$
or, equivalently:
$$
\begin{equation}
\begin{split}
\forall \vec{x} \in H: h(\vec{x}) \cdot (x - f_A(\vec{x})) \cdot (x - f_B(\vec{x})) - \\
(x - f_B(\vec{x})) + m(\vec{x}) \cdot (x - f_A(\vec{x}) = 0
\end{split}
\end{equation}
$$
So, we:
- Apply the [[Multivariate Sum-Check Protocol]] to show that $\sum_{\vec{x} \in H} h(\vec{x}) = 0$
- Apply the [[Multivariate Zero Check Protocol]] to prove the identity above

The two protocols can be run in one go, as a single sum-check applied to:
$$
\begin{equation}
\begin{split}
h(\vec{x}) + &eq(\vec{z}, \vec{x}) \cdot( \\
&h(\vec{x}) \cdot (x - f_A(\vec{x})) \cdot (x - f_B(\vec{x})) - \\
&(x - f_B(\vec{x})) + m(\vec{x}) \cdot (x - f_A(\vec{x}))
\end{split}
\end{equation}
$$
where $\vec{z}$ is a verifier challenge and $eq$ is the unique multilinear extension of the equality function on the Boolean hypercube.
#### Batching
In principle, this technique can be generalized to looking up many columns, by defining $h$ such that:
$$
\forall \vec{x} \in H: h(\vec{x}) = (\sum_{i}\frac{1}{x - f_{A_i}(\vec{x})}) - \frac{m(\vec{x})}{x - f_B(\vec{x})}
$$
The problem with this is that the degree of the zero-check expression grows linearly with each column. The paper describes a way to split $h$ into $h_1, ..., h_K$ such that $h = \sum_ih_i$. This decreases the degree, but requires the prover to commit to more polynomials. The ideal value for $K$ depends on the polynomial commitment scheme being used.
### Fractional sum-check via GKR
([[Improving logarithmic derivative lookups using GKR]])

With this approach, we get rid of the need to commit to the "helper polynomials" $h_i$.

Let $f, m: \mathbb{F}^n \rightarrow \mathbb{F}$ be the *concatenations* of the table values and multiplicities ($-1$ for each column on the LHS of the lookup, otherwise the combined multiplicity). We essentially want to show that:
$$
\sum_{\vec{x} \in H} \frac{m(\vec{x})}{f(\vec{x})} = 0
$$
Adding to rational numbers $\frac{a_0}{b_0}$ and $\frac{a_1}{b_1}$ can be achieved by mapping $(a_0, b_0)$ and $(a_1, b_1)$ to $(a_0 \cdot b_1 + a_1 \cdot b_0, b_0 \cdot b_1)$. So, we can compute the sum as a layered arithmetic circuit of depth $O(\log n)$ by explicitly keeping track of the numerators and denominators and summing them up pair-wise in each layer!

The execution of the circuit can then be proven using the [[GKR]] protocol.
### cq (cached quotients)
cq is essentially an instantiation of LogUp (in its original variant) in the univariate setting, using [[KGZ]] commitments.

The main achievement is that the prover time is independent of the table size: We can prove a subset relation for a multi-set of size $n$ into a table of size $N$ in time $O(n \log n)$ (and pre-processing time $O(N \log N)$).

In a nutshell:
- The prover commits to polynomials $f_A$, $f_B$, and $m$
- The verifier sends a random challenge $\beta$
- Now, the prover computes and commits to $h_A$ and $h_B$ such that:
	- $\forall x \in H: h_A(x) \cdot (f_A(x) + \beta) - 1 = 0$
	- $\forall x \in H: h_B(x) \cdot (f_B(x) + \beta) - m(x) = 0$
	- $\sum_{x \in H} h_A(x) = \sum_{x \in H} h_B(x)$

Note that:
- The degree of $f_A$ and $h_A$ is $n$; the degree of $f_B$ and $h_B$ is $N$
- However, because $f_B$ is $n$-sparse, its commitment can be computed in $O(n)$ time, by adding pre-computed commitments to the Lagrange polynomials
- The main issue is that the [[Univariate Zero-Check Protocol]] requires computing a *commitment* to an $N$-degree quotient polynomial $q$ such that $h_B(X) \cdot (f_B(X) + \beta) - m(X) = q(X) \cdot Z_{H}(X)$

The key insight is that the quotient $q$ is also $n$-sparse (but in a different basis) and can be computed in $O(n)$ time!

First, note that the $q$ we're computing is the same quotient that we get from dividing $h_B(X) \cdot f_B(X)$ by $Z_H(X)$ with remainder:
$$
h_B(X) \cdot f_B(X) = q(X) \cdot Z_H(X) + r(X)
$$
(where it happens that $r(X) = m(X) - \beta \cdot h_B(X)$)

Second, because $h_B$ is just a linear combination of Lagrange polynomials, we can write $q$ as the *same* linear combination of polynomials $q_i$ where:
$$
L_i(X) \cdot f_B(X) = q_i(X) \cdot Z_H(X) + r_i(X)
$$
Note that the quotients $q_i$ are independent of $A$! Therefore, we can pre-compute their commitments once and use them to compute the commitment to $q$ in $O(n)$ time.
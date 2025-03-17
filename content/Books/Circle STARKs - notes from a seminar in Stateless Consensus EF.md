Author: Matan Prasma
PDF: can be found [here](https://sites.google.com/view/matanprasmashomepage/publications)
Content is also from online lectures

# Naive Set Theory (Elliptic Curve Script)
- **Ordered pair**: $(a, b) := \{\{a\}, \{a, b\}\}$
- **Cartesian product**: $A \times B := \{(a, b) | a \in A, b \in B\}$
- **Function**: A function $f: A \rightarrow B$ is a subset $f \subset A \times B$ s.t.:
	- $\forall a \in A: \exists b \in B: (a, b) \in f$
	- $\forall (a, b) \in f, (a, b') \in f: b = b'$
	- => $A$ is the *domain*, $B$ is the *range*
- **Function composition**: $(g \circ f)(a) := g(f(a))$
- **Function properties**:
	- **Injective**, Monomorphism, "one-to-one":
	  $\forall x, x': f(x) = f(x') \implies x = x'$
	- **Surjective**, Epimorphism, "onto":
	  $\forall y: \exists x: f(x) = y$
	- **Bijective**, Isomorphism, "invertible": Injective + Surjective
	- => Each of these properties still hold for $(g \circ f)$ if they do for $f$ and $g$
- **Equivalence relations**: A relation $R$ on $X$ is a subset $R \subset X \times X$
	- $x \sim_R y$ means that \mathbb{Z}_b$(x, y) \in R$
	- An *equivalence relation* is a relation with the properties:
		- Reflexivity: $\forall x \in X: x \sim x$
		- Symmetry: $\forall x, y \in X: x \sim y \iff y \sim x$
		- Transitivity: $\forall x, y, z \in X: x \sim y \wedge y \sim z \implies x \sim z$
	- **Equivalence class**: $[x] := \{y \in X| x \sim y\}$
	- **Quotient set**: $X / \sim := \{[x] | x \in X\}$
		- The *quotient map* maps an element $x$ to its equivalence class $[x]$
	- **Example: Projective Line**:
		- On some field $\mathbb{F}$, define the **punctured plane** as $\mathbb{F}^2 \setminus \{(0, 0)\}$
		- Equivalence relation:
		  $(x, y) \sim (x', y') \iff \exists \lambda \neq 0: (x, y) = (\lambda x', \lambda y')$
		- The quotient set is the **projective line** and denoted $\mathbb{P}_{\mathbb{F}}^1$
		- Elements of this set are denoted as $[x:y]$
		- The number of elements is $|\mathbb{F}| + 1$: One element for each possible slope, plus the "point at infinity" ($[0:1]$)
# Elementary Number Theory
- **Bezout's lemma**: For any positive integers $a, b$, there exist $u, v \in \mathbb{Z}$ such that:
  $au + bv = gcd(a, b)$
	- => These numbers can be found using the [Extended Euclidean Algroithm](https://en.wikipedia.org/wiki/Extended_Euclidean_algorithm)
	- => One application: If $a < b$ and $b$ is prime, we have $gcd(a, b) = 1$, so $u$ is the multiplicative inverse of $a$ in $\mathbb{Z}_b$
- **Chinese Remainder Theorem**: 
	- Let $n_1, \dots, n_k \in \mathbb{N}$ be such that $\forall i \neq j : \gcd(n_i, n_j) = 1$ and denote  $N = n_1 \cdot \dots \cdot n_k$.  
	- Then for all $a_1, \dots, a_k \in \mathbb{N}$, *the system of equations*$$
\begin{aligned}
    x &= a_1 \pmod{n_1} \\
      &\vdots \\
    x &= a_k \pmod{n_k}
\end{aligned}
$$*has a unique solution in* $\mathbb{Z}_N$.
# Groups
- **Group**: A group consists of a set $G$, containing a *neutral element* $e$, and a *group operation* $\mu$ under the following conditions:
	- **Unitary**: $\forall x \in G: \mu(x, e) = \mu(e, x) = x$
	- **Associativity**: $\forall x, y, z \in G: \mu(\mu(x, y), z) = \mu(x, \mu(y, z))$
	- **Invertibility**: $\forall x \in G: \exists x^{-1} \in G: \mu(x^{-1}, x) = \mu(x, x^{-1}) = e$
- Common notation:
	- Additive: $e$ is denoted as $0$; $\mu$ is denoted as $+$
	- Multiplicative: $e$ is denoted as $1$; $\mu$ is denoted as $\cdot$
- **Fermat's little theorem**: If $p$ is prime, $\forall a \in \{1, ..., p - 1\}: a^{p - 1} = 1 \mod p$
- => This implies that $\mathbb{F}_p^{\times} = \mathbb{F} \setminus \{0\} = \{1, ..., p - 1\}$ is a group with multiplication modulo $p$
- **Subgroup**: If a for subset $H \subseteq G$ $(H, \mu, e)$ is a group, it is called a *subgroup* of $G$ (denoted $H \leq G$)
- **Coset**: Given a subgroup $H$ and an element $x \in G$, the (left) $H$-coset is defined as:
  $xH := \{xh | x \in H\}$
	- $xH = yH \iff x^{-1}y \in H$
	- The set of cosets $\{xH\}_{x \in G}$ forms a partition of $G$
- **Lagrange's Theorem**: For a finite group $G$ and subgroup $H \le G$, $\#H$ divides $\#G$
	- (Because each coset has $\#H$ elements and the set of cosets is a partition of $G$)
- **Group Homomorphism**: For two groups $G$ and $H$, a function $f: G \rightarrow H$ is called a group homomorphism if for any $x, y \in G$, $f(xy) = f(x) \cdot f(y)$
	- The **kernel** $ker(f)$ is defined as $\{x \in G | f(x) = e_H\}$
	- The **image** $Im(f)$ is defined as $\{f(x) | x \in G\} \subseteq H$
- If $f$ is bijective, it is called a **group isomorphism** and we denote $G \cong H$
- The **quotient group** $G / H = \{xH | x \in G\}$ with $xH \cdot yH = (xy) H$ is a group and the **quotient map** $q: G \rightarrow G / H$ given by $x \mapsto xH$ is a group homomorphism
- **Cyclic groups**:
	- For a group $G$ and element $g \in G$, the **order** of $g$ $o(g)$ is the minimal natural number $n$ such that $g^n = e$. If no such number exists, we set $o(g) = \infty$.
	- **Cauchy's theorem**: For a finite abelian group $G$ and prime $p$, if $p \mid \#G$, there exists an element $g \in G$ with $o(g) = p$
		- (partial converse of Lagrange's theorem)
	- A group $G$ is called **cyclic** if there exists an element $g \in G$ such that *the group generated by $g$, $\langle g \rangle := \{g^n \mid n \in \mathbb{Z}\}$, equals $G$
	- Any cyclic group is **abelian** and **isomorphic to $\mathbb{Z}_{\#G}$** (because any element can be expressed uniquely as $g^{i}$ for $i \in \mathbb{Z}_{\#G}$)
	- **Euler totient function**: $\varphi(n) = \#\{ k \mid 1 \leq k \leq n \wedge \gcd(n, k) = 1 \}$
	- An element in $k \in \mathbb{Z}_n$ is a generator if $gcd(k, n) = 1$, and  $\varphi(n)$ gives the number of generators of $\mathbb{Z}_n$
	- **Fundamental Theorem of Cyclic Groups**: For a cyclic group $G$
		- Any subgroup is cyclic
		- For any $k \mid \#G$, $\langle g^{\#G / k} \rangle$ is the unique subgroup of size $k$ 
	- Lagrange's theorem implies that any prime-order group is cyclic!
# Fields
- A **field** is a set $\mathbb{F}$ with binary operations $+: \mathbb{F} \times \mathbb{F} \rightarrow \mathbb{F}$ and $\cdot: \mathbb{F} \times \mathbb{F} \rightarrow \mathbb{F}$ and elements $0 \neq 1 \in \mathbb{F}$ such that:
	- Both $(\mathbb{F}, +, 0)$ and $(\mathbb{F} \setminus \{0\}, \cdot, 1)$ are abelian groups
	- For every $x, y, z \in \mathbb{F}, x \cdot (y + z) = x\cdot y + x\cdot z$
- The field **characteristic** is the minimal number $n$ such that $1$ added to itself $n$ times is $0$ (or $0$, if no such number exists)
- A **subfield** needs to contain $0$ and $1$ and be closed under the two operations.
- **Polynomials** over a field:
	- **Polynomial** in one variable: a formal expression of the form $f(x) = \sum_{i = 0}^n a_i \cdot x^i$ where $a_i \in \mathbb{F}$ are the **coefficients**, the terms $a_i \cdot x^i$ are called **monomials**, $\deg f := n$ is the **degree** and the set of all polynomials is denoted as $\mathbb{F}[x]$.
	- Polynomials can be added, subtracted, multiplied and divided. Division example:![[Screenshot 2025-02-27 at 10.06.27.png]]
	- => Polynomials form a **ring**: It's like a field, but not all polynomials have multiplicative inverses
	- A polynomial as a root $\alpha$ iff. it is divisible by $(x - \alpha)$
		- => a polynomial can have at most $n$ roots
	- A **prime (or irreducible) polynomial** is one that cannot be expressed as a product of two non-constant polynomials.
	- **Monic** polynomial: Leading coefficient is 1.
- **Modulo algorithmic of polynomials**: For a prime polynomial $\phi$ over a field $\mathbb{F}$, we can define a field $\mathbb{F}_{\phi(x)} = \mathbb{F}[x] / (\phi)$ => $\phi$ does not have a root in $\mathbb{F}$, but in $\mathbb{F}_{\phi(x)}$ it has a root: $x$
- A **splitting field** for a polynomial $f$ is one where $f$ can be expressed as a product of linear factors which is minimal, i.e., there is no subfield with the same property.
	- It always exists: we can just repeat extending the field by non-linear prime polynomials until we have none left.
	- If $\mathbb{F}$ is a finite field of size $q = p^n$ (for a prime $p$ and some natural number $n$), it is a splitting field of $x^q-x$
- **Galois fields**: For any prime $p$ and natural number $n$, there exists a finite field of size $p^n$. Conversely, any finite field has a size of the form $p^n$, and all fields of the same size are isomorphic. Therefore, all finite fields can be denoted as $GF(p^n)$.
- **Algebraic Closure**: For a finite field $\mathbb{F} = \mathbb{F}_q$ for $q = p^m$, the algebraic closure $\overline{\mathbb{F}} = \bigcup_{n \in \mathbb{N}} \mathbb{F}_{q^{n}}$ is a infinite-size field with characteristic $q$ where all polynomials split.
	- To add or multiply two elements from $\mathbb{F}_{n_1}$ and $\mathbb{F}_{n_2}$, embed them into $\mathbb{F}_{n_1 \cdot n_2}$ and do the operation there
- The **$n$th roots of unity** $\mu_n$ on a prime field $\mathbb{F}_{p}$ is the sets of roots of $x^n-1$ in $\overline{\mathbb{F}}_p$
	- If $p$ does not divide $n$, $\mu_n$ is a cyclic group
- **Lagrange interpolation**: Given points $(x_0, y_0), ... (x_n, y_n) \in \mathbb{F} \times \mathbb{F}$, the ith Lagrange polynomial is $L_i(x) := \frac{\prod_{j \neq i} x - x_j}{\prod_{j \neq i} x_i - x_j}$ and the Lagrange interpolation of the points is $L(x) := \sum_{i = 0}^n y_i \cdot L_i(x)$
- **Fast Fourier Transform** (FFT):
	- Polynomials can be represented in evaluation form ("sampling") and coefficient form ("vector")
	- The FFT is an efficient algorithm to compute the *Discrete Fourier Transform (DFT)*, which is a matrix-vector product of the **Vandermonde matrix** with the polynomial's coefficients => performs **evaluation**
		- *TODO: Put matrix*
	- Algorithm: ![[Screenshot 2025-03-04 at 20.41.46.png]]
# Errors?
- Example 5.15: How can 2 times 2 be 0? Then $(\mathbb{F} \setminus 0, \cdot, 1)$ would not be a group?
- Example 5.60: The factor should be $(x^2 + \sqrt{2})$?
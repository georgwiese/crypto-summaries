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
	- $x \sim_R y$ means that $(x, y) \in R$
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
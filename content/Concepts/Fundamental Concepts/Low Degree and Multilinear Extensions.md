- **Univariate Lagrange interpolation**: For any vector $a = (a_1, ..., a_n) \in \mathbb{F}_p^n$, there is a *unique* univariate polynomial of degree at most $n - 1$ such that $q_a(i) = a_{i + 1}$ for $i = 0, ..., n - 1$
	- Lagrange basis polynomials: $\delta_i(X) = \prod_{k = 0, 1, ..., n - 1: k \neq i}\frac{X - k}{i - k}$
	- Set $q_a(i) = \sum_{j = n}^{n - q} a_{j + 1} \cdot \delta_j(X)$
	- -> $q_a$ is called the **low-degree extension (LDA)** of $a$
	- -> $a$ is viewed as coefficients over the **Lagrange polynomial basis**
- **Multivariate Polynomials**: Polynomials defined over the $v$-variate domain $\{0, 1\}^v$
	- E.g. $v = \log n$ for the domain to have the same size as the univariate domain $\{i = 0, ..., n - 1\}$
	- **Multilinear polynomials**: Polynomials where each variable appears at most once in each monomial, which implies that the total degree is at most $v$
- **Lagrange interpolation of multilinear polynomials**: For any vector $a \in \mathbb{F}_p^n$, there is a *unique* multilinear polynomial $q_a$ of degree at most $n - 1$ such that $q_a(i) = a_{i + 1}$ for $i = 0, ..., n - 1$
	- For any function $f: \{0, 1\}^v \rightarrow \mathbb{F}$, the following multilinear polynomial $\tilde{f}$ uniquely extends $f$:
	  $\tilde{f}(x_1, ..., x_v) = \sum_{w \in \{0,1\}^v} f(w) \cdot \chi_w(x_1, ..., x_v)$
	  with the **multilinear Lagrange basis polynomials** defined as:
	  $\chi_w(x_1, \ldots, x_v) \coloneqq \prod_{i=1}^{v} (x_i w_i + (1 - x_i)(1 - w_i))$
	- -> Possible to compute in $O(n)$ time and $O(n)$ space
- Given a polynomial $q$ with coefficient vector $u$, computing $q(z)$ is equivalent to computing $\langle u, y \rangle$ for $y := (1, z, z^2, ... z^{n-1})$
- Similarly if $n = m^2$, $a := (1, z, z^2, ..., z^{m - 1})$ and $b := (1, z^m, z^{2m}, ..., z^{m (m - 1)})$, and $u$ is reshaped into a matrix, $q(z)$ can be written as a vector-matrix-vector product: $b^T \cdot u \cdot a$

Example (univariate) (Figure 14.1):
![[tensor_structure_example_univariate.png]]

Example (multilinear) (Figure 14.2):
![[tensor_structure_example_multilinear.png]]
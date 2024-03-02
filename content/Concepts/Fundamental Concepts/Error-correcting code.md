An **error-correcting** code $E$ maps vectors in $\mathbb{F}^m$ to slightly longer vectors in $\mathbb{F}^{\rho^{-1} \cdot m}$, where $\rho$ is the **rate** of the code (e.g. $1 / 4$). It should be **distance-amplifying**, which means that if two words are different, their code should disagree in a large fraction of coordinates.
A code is **linear** if $E(a \cdot u_1 + b \cdot u_2) = a \cdot E(u_1) + b \cdot E(u_2)$
- e.g. Reed-Solomon code: View $u$ as coefficients of a degree-$m$ polynomial
A code is **systematic** if the first $m$ symbols of $E(u)$ are the entries of $u$ itself
- e.g. univariate low-degree extension code, see [[Low Degree and Multilinear Extensions]]
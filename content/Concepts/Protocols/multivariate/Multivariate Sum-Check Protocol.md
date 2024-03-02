Given a $v$-variate polynomial $g$ defined over field $\mathbb{F}$, the protocol convinces the verifier that the she computed the following some correctly:
$$
H := \sum_{b_1 \in \{0, 1\}} \sum_{b_2 \in \{0, 1\}} ... \sum_{b_v \in \{0, 1\}} g(b_1, b_2, ..., b_v)
$$
The advantage of using the protocol is that the verifier runtime is only $O(v)$ plus the time to evaluate $g$ at a single input in $\mathbb{F}^v$.
![[multivariate_sum_check.png]]
Source: [The Halo2 Book - Lookup argument](https://zcash.github.io/halo2/design/proving-system/lookup.html)

To show that $A \subseteq B$, we let the prover compute columns $A'$ and $B'$ such that:
- $A'$ is a permutation of $A$
- $B'$ is a permutation of $B$
- In each row:
	- $A'_{i + 1} = A'_i$, OR
	- $A'_{i + 1} = B'_i$

**Example**:

| A | B | A' | B' |
|---|---|----|----|
| 1 | 4 | 1  | 1  |
| 1 | 3 | 1  | 2  |
| 4 | 2 | 4  | 4  |
| 4 | 1 | 4  | 3  |
For the permutation argument, they do a [[Permutation Check via Product Check]], using a *single* accumulator column $Z$ for both permutations.
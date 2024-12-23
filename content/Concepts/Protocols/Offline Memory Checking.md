by [Blum at al. (1994)](https://link.springer.com/article/10.1007/BF01185212)

Sources:
- [Explanation (YouTube)](https://youtu.be/dmVweFbJsxw?si=26si_BTIIcVzPb0m&t=4474)
- [[Unlocking the lookup singularity with Lasso]]

### Read-only memory
The problem they tried to solve is to force an untrusted memory to prove it worked correctly, with very low space requirements of the verifier.

The verifier maintains two sets, which can be compressed and updated incrementally via permutation-invariant fingerprinting (see [[Permutation Check via Product Check]]):
- **Read Set**: The tuples $(address_i, value_i, count_i)$ returned by the prover
- **Write Set**: $(address_i, value_i, count_i)$ written to memory

Then, the verifier behaves as follows:
- At the beginning of the execution, they write for each address $a_i$ the tuple $(a_i, 0, 0)$ (or some other initial value)
- Each read returning $(a_i, v_i, c_i)$ is followed by a write of $(a_i, v_i, c_i + 1)$
- At the end of the execution, the verifier reads all addresses

If and only if the prover is honest, the read and write *multi*-sets will be equal.

### Application: Indexed Lookup
Source: [[Unlocking the lookup singularity with Lasso]]

Suppose we have:
- A multi-linear extension $t: \mathbb{F}^{\log M} \rightarrow \mathbb{F}$  of some table (i.e., read-only memory)
- A multi-linear extension $a: \mathbb{F}^{\log m} \rightarrow \mathbb{F}$ of addresses
- A multi-linear extension $v: \mathbb{F}^{\log m} \rightarrow \mathbb{F}$ of values

We want to show that:
$$
\forall{j \in \{0, 1\}}^{\log m}: v(j) = t(\mathtt{bits}(a(j)))
$$
where $\mathtt{bits}: \mathbb{F} \rightarrow \mathbb{F}^{\log N}$  is the function that maps a field element to its bit representation.

We can show this is the case by simulating the offline memory verifier in a SNARK.

Specifically, the prover commits to two polynomials, $c_{read}: \mathbb{F}^{\log m} \rightarrow \mathbb{F}$ and $c_{final}: \mathbb{F}^{\log M} \rightarrow \mathbb{F}$, containing the counts returned by the offline memory prover for the "normal" and final read operations.

The memory operations were performed correctly if $RS = WS$ where:
$$
\begin{equation}
\begin{split}
RS = &\{(a(j), v(j), c_{read}(j)): j \in \{0, 1\}^{\log m}\} \\ &\cup \{(\mathtt{to\_field}(j), t(j), c_{final}(j)): j \in \{0, 1\}^{\log M}\}
\end{split}
\end{equation}
$$
$$
\begin{equation}
\begin{split}
WS = &\{(\mathtt{to\_field}(j), t(j), 0): j \in \{0, 1\}^{\log M}\} \\ &\cup \{(a(j), v(j), c_{read}(j) + 1): j \in \{0, 1\}^{\log m}\}
\end{split}
\end{equation}
$$
where $\mathtt{to\_field}$ is also a multi-linear polynomial:
$$
\mathtt{to\_field}(x_1, ..., x_{\log M}) := \sum_{i = 1}^{\log M} 2^{i - 1} \cdot x_i
$$

The two multi-sets can be shown to be equal using the [[Permutation Check via Product Check]]! In particular, the grand product can be computed using a layered arithmetic circuit of depth $O(\log m + \log M)$ and proven using [[GKR]].
#### PIL Sketch
```rs
namespace main;
  // Address might be given (indexed lookup)
  // or has to be provided as a witness.
  col fixed address;
  col witness value;
  col witness time_stamp;

  // Read value
  bus_receive(1, (address, value, time_stamp));
  // Write value
  bus_send(1, (address, value, time_stamp + 1));

namespace table;
  // Often, this does not need to be committed
  // but corresponds to a polynomial that can be
  // evaluated cheaply by the verifier, e.g.:
  // f(x_0, ..., x_{n - 1}) =
  //   x_0 + ... + 2^{n - 1} * x^{n - 1}
  col fixed address;
  col fixed value;
  // The number of times each element is read, i.e.,
  // the final time steps.
  col witness multiplicities;

  // Initialize memory: Write all values with
  // time step 0
  bus_send(1, (address, value, 0));

  // Finalize memory: Read all values one last time
  bus_receive(1, (address, value, multiplicity));
```

Comparison to LogUp (see [[LogUp & cq]]):
- The multiplicity with which elements are sent to the bus (the first argument of `bus_send` and `bus_receive`) is always 1.
- As a consequence, the bus argument does not need to support multiplicities $> 1$ and can be simply a [[Permutation Check via Product Check]]. This is cheaper than the fractional sum-check needed in LogUp.
- On the other hand, LogUp commits to a strict subset of columns: It does not need the `time_step` column. Also, in the case of non-indexed lookups (simply stating that `main::value` is a subset of `table::value`), the `address` columns are not needed.
### Read-write memory
The original paper actually describes a read-write memory which is slightly more complex:
- As described above, all memory cells are initialized with some value at time step $t = 0$
- For each memory write of value $v_{cur}$ to address $addr$ at time $t_{cur}$:
	- The prover provides $t_{prev}$ and $v_{prev}$
	- The verifier asserts that $t_{prev} < t_{cur}$
	- The tuple $(addr, t_{prev}, v_{prev})$ is added to the read set
	- The tuple $(addr, t_{cur}, v_{cur})$ is added to the write set
- A memory read works the same, but with $v_{prev} = v_{cur}$
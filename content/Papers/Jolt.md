[Jolt Book](https://jolt.a16zcrypto.com/intro.html) | [ePrint 2023/1217](https://eprint.iacr.org/2023/1217) | a16z, 2023
Authors: Arasu Arun, Srinath Setty, Justin Thaler

Jolt is a RISC-V zkVM built around sum-check and lookup arguments: **J**ust **O**ne **L**ookup **T**able. As of v0.2.0, it uses [[Twist and Shout|Twist & Shout]] for memory checking / lookups, [[Spartan]] for R1CS, and [[Dory]] as the polynomial commitment scheme. It supports **RV64IMAC**.

## Instruction Fetch (Bytecode)

*Source: [Jolt Book - Bytecode](https://jolt.a16zcrypto.com/how/architecture/bytecode.html)*

![[bytecode.png]]

During preprocessing, the ELF binary is decoded into a table of instructions. Each entry is a tuple:

`(rs1, rs2, rd, imm, circuit_flags, lookup_table_flags, address)`

where:
- **rs1, rs2**: Source register indices (which of the 64 registers to read).
- **rd**: Destination register index (which register to write the result to).
- **imm**: Immediate value embedded in the instruction.
- **address**: The instruction's ELF memory address (`unexpanded_pc`). This is distinct from the bytecode *index* $k$ (its position in the preprocessed table).
- **circuit_flags**: Booleans consumed by the R1CS constraints. They encode the instruction *type* for the constraint system. Examples:
  - `jump_flag` (JAL, JALR)
  - `branch_flag`
  - `load_flag` (LB, LW, ...)
  - `add_operands` (ADD, ADDI, AUIPC — instructions whose operands are added in the field)
  - `is_rd_not_zero`, `write_lookup_to_rd`
  - `left_is_rs1`, `left_is_pc`, `right_is_rs2`, `right_is_imm` (which values to feed as the left/right operand)
  - `is_noop`, `virtual_instruction`, `is_first_in_sequence`
- **lookup_table_flags**: Booleans that select *which* instruction lookup table to query during instruction execution (see below). There is one flag per supported instruction (e.g. one for XOR, one for AND, one for SLT, ...).

At each cycle, the current PC indexes into this table to "fetch" the instruction. This is a read-only lookup, so it uses a [[Twist and Shout|Shout]] instance. The $\mathsf{Val}$ polynomial is a random linear combination of all the fields above, so that a single read-check proves the entire tuple at once.

## Registers

*Source: [Jolt Book - Registers](https://jolt.a16zcrypto.com/how/architecture/registers.html)*

Jolt has 64 registers (32 real RISC-V + 32 virtual):

| Registers | Purpose                                                   |
| --------- | --------------------------------------------------------- |
| 0–31      | Standard RISC-V registers (x0 hardwired to zero)          |
| 32–33     | LR/SC reservation addresses (atomics)                     |
| 34–39     | M-mode Control and Status Registers (CSRs), handles traps |
| 40–46     | Temporaries for virtual instruction sequences             |
| 47–63     | Temporaries for inline sequences                          |

Register read/write correctness is proven using [[Twist and Shout|Twist]] with $K = 64$ and $d = 1$. Each cycle reads up to two source registers (rs1, rs2) and writes one destination (rd), so there are two $\mathsf{ra}$ polynomials and one $\mathsf{wa}$ polynomial, all batched into a single sum-check instance.

**No separate one-hot checks for registers**: Normally in [[Twist and Shout|Shout/Twist]], the prover commits to $\mathsf{ra}_i$ polynomials and must prove they're one-hot. For registers this is unnecessary: which register gets accessed is already determined by the instruction's rs1/rs2/rd fields, and the bytecode Shout already proves those. So the register $\mathsf{ra}$ at a random point can be derived via a sum-check over the bytecode's $\mathsf{ra}$ and the register index fields — no separate commitment or one-hot proof needed.

## Instruction Execution

*Source: [Jolt Book - Instruction Execution](https://jolt.a16zcrypto.com/how/architecture/instruction_execution.html)*

The key insight of Jolt: **instruction execution is a single giant lookup**. For each instruction, the two 64-bit operands $(x, y)$ form the lookup index, and the table returns the correct output. Their bits are interleaved:
$$
(k_1, k_2, \ldots, k_{128}) = (x_1, y_1, x_2, y_2, \ldots, x_{64}, y_{64})
$$

This gives a table of size $K = 2^{128}$, which obviously cannot be materialized.

### Prefix-Suffix Decomposition

The trick: for many instructions, the table's MLE has **prefix-suffix structure** (Appendix A of [Proving CPU Executions in Small Space](https://eprint.iacr.org/2025/105)). A multilinear polynomial $\widetilde{a}(x_1, \ldots, x_n)$ has prefix-suffix structure for cutoff $i$ with $k$ terms if:
$$
\widetilde{a}(x_1, \ldots, x_n) = \sum_{j=1}^{k} \text{prefix}_j(x_1, \ldots, x_i) \cdot \text{suffix}_j(x_{i+1}, \ldots, x_n)
$$

The **prefix-suffix inner product protocol** exploits this to run the sum-check $\sum_x \widetilde{u}(x) \cdot \widetilde{a}(x)$ (where $u$ is the sparse one-hot address) without ever materializing the $2^{128}$-entry table. It proceeds in $C$ stages, each handling $n / C$ sum-check rounds:
1. At each stage, the prover makes a **single pass** over the sparse $u$ to build a small array $Q$ of size $N^{1/C}$ (aggregating suffix contributions).
2. It builds a small array $P$ of size $N^{1/C}$ (prefix evaluations).
3. The sum-check for those $n/C$ rounds reduces to an inner product $\widetilde{P}(y) \cdot \widetilde{Q}(y)$, which is only $N^{1/C}$-dimensional.

For this to work, $\widetilde{a}$ must have prefix-suffix structure at **every** cutoff $n/C, 2n/C, \ldots, (C{-}1)n/C$ (not just one). The total prover cost is $O(C \cdot k \cdot m)$ where $m$ is the sparsity of $u$ (= number of trace cycles $T$).

### Example: SLT (Set Less Than)

SLT maps $(x, y) \mapsto 1$ if $x < y$, else $0$. Recall the index is interleaved: $k = (x_1, y_1, x_2, y_2, \ldots)$. With a cutoff after 16 bits (8 bit-pairs), $k_{\text{prefix}}$ contains the high bits $(x_1, y_1, \ldots, x_8, y_8)$ and $k_{\text{suffix}}$ the low bits $(x_9, y_9, \ldots, x_{64}, y_{64})$.

The comparison decomposes as: $x < y$ iff the high bits of $x$ are less than those of $y$, OR the high bits are equal and the low bits of $x$ are less than those of $y$:

$$
\widetilde{\mathsf{Val}}_{\text{SLT}}(k_{\text{prefix}}, k_{\text{suffix}}) = \text{LT}_{\text{high}}(k_{\text{prefix}}) \cdot 1 + \text{EQ}_{\text{high}}(k_{\text{prefix}}) \cdot \text{LT}_{\text{low}}(k_{\text{suffix}})
$$

This is $k = 2$ terms. Term 2 has non-trivial factors on **both** sides: whether the suffix matters is gated by the prefix bits being equal. This structure holds at every cutoff boundary (it's the standard recursive definition of lexicographic comparison), so it works for any $C$.

### Example: XOR

XOR is a simpler case. Each output bit depends on a single input bit-pair independently: $x_i \oplus y_i = x_i + y_i - 2x_i y_i$. So the decomposition is purely additive with no cross-boundary interaction:

$$
\widetilde{\mathsf{Val}}_{\text{XOR}}(k_{\text{prefix}}, k_{\text{suffix}}) = \text{prefix}_{\text{XOR}}(k_{\text{prefix}}) \cdot 1 + 1 \cdot \text{suffix}_{\text{XOR}}(k_{\text{suffix}})
$$

where $\text{prefix}_{\text{XOR}}$ sums the weighted per-bit XORs for bits in the prefix, and $\text{suffix}_{\text{XOR}}$ does the same for the suffix. This is $k = 2$ but both terms have a trivial factor (constant 1 on one side).

### Multiplexing Between Instructions

Since different instructions have different tables, a boolean **lookup table flag** $\mathsf{flag}_\ell(j)$ (fetched from the bytecode) indicates which table is active at cycle $j$. The multiplexed read-checking sum-check becomes:

$$
\widetilde{\mathsf{rv}}(r_{\text{cycle}}) = \sum_{k, j} \widetilde{\mathsf{eq}}(r_{\text{cycle}}, j) \cdot \left(\prod_{i=1}^{d} \widetilde{\mathsf{ra}}_i(k_i, j)\right) \cdot \left(\sum_\ell \mathsf{flag}_\ell(j) \cdot \widetilde{\mathsf{Val}}_\ell(k)\right)
$$

In practice, $d = 16$ for instruction execution, giving $K^{1/d} = 2^{128/16} = 2^8 = 256$ (i.e., each of the 16 one-hot chunks has 256 entries). The sum-check degree per round is $d + 1 = 17$. Jolt uses techniques from Karatsuba/Toom-Cook to optimize the degree-17 polynomial evaluations.

### Recovering Dense Operands (raf)

The R1CS constraints need the *dense* operand values (as field elements), not one-hot encodings. These are recovered via **raf-evaluation sum-checks** (see [[Twist and Shout]], "Recovering dense addresses"]]). For two interleaved operands:

$$
\mathsf{LeftOperand}(r) = \sum_{k,j} \widetilde{\mathsf{eq}}(r,j) \cdot \widetilde{\mathsf{ra}}(k,j) \cdot \sum_{\ell=0}^{\log(K)/2-1} 2^\ell \cdot k_{2\ell}
$$

$$
\mathsf{RightOperand}(r) = \sum_{k,j} \widetilde{\mathsf{eq}}(r,j) \cdot \widetilde{\mathsf{ra}}(k,j) \cdot \sum_{\ell=0}^{\log(K)/2-1} 2^\ell \cdot k_{2\ell+1}
$$

These extract the even-indexed bits (left operand $x$) and odd-indexed bits (right operand $y$) respectively. The resulting dense values are fed into the R1CS constraints for linking.

For instructions with a single operand (e.g., range checks), the index is not interleaved: the first 64 bits are zero-padded, and only `RightOperand` is non-trivial.

## Committed Polynomials (according to Claude)

All committed polynomials are opened via [[Dory]], which has "pay-per-bit" costs: boolean entries (the one-hot 1s) are much cheaper to commit than full field elements.

| Polynomial | Component | Hypercube size | Non-zero entries | Entry type |
|---|---|---|---|---|
| $\widetilde{\mathsf{ra}}_1, \ldots, \widetilde{\mathsf{ra}}_{16}$ | Instruction exec (Shout) | $2^8 \times T$ each | $T$ each (sparse) | Boolean |
| $\widetilde{\mathsf{ra}}_1, \ldots, \widetilde{\mathsf{ra}}_d$ | Bytecode (Shout) | $P^{1/d} \times T$ each | $T$ each (sparse) | Boolean |
| $\widetilde{\mathsf{ra}}_1, \ldots, \widetilde{\mathsf{ra}}_d$ | RAM (Twist) | $K^{1/d} \times T$ each | $\leq T$ each (sparse) | Boolean |
| $\widetilde{z}$ | Spartan (R1CS witness) | $W \times T$ | Dense | Field elements |
| $\widetilde{\mathsf{Inc}}$ | Registers (Twist) | $T$ | Dense | Field elements |
Parameters:
- **Instruction execution**: $d = 16$, $K^{1/d} = 2^8 = 256$. This is the largest commitment: $16T$ boolean entries total.
- **Bytecode**: $d$ depends on program size $P$ (not a fixed constant).
- **RAM**: $K^{1/d} = 2^4$ or $2^8$ depending on $T$. Only $\mathsf{ra}$ (no separate $\mathsf{wa}$) since RV64IMAC does at most one memory operation per cycle.
- **Spartan**: $W$ is the number of R1CS variables per cycle (exact value unknown).
- **Advice**: Not separately committed polynomials. They are folded into the RAM initial state polynomial $\mathsf{ram\_init}$. "Trusted" advice has an externally-generated commitment; "untrusted" advice is committed by the prover. Both occupy the lowest addresses in the RAM table.

### Uncertain / needs verification

- **Register $\mathsf{ra}$ / $\mathsf{wa}$**: The register addresses are derived from bytecode (see "no separate one-hot checks" above). This suggests they are **virtual** (not committed), meaning the register Twist only commits to $\mathsf{Inc}$. But the docs are not fully explicit.
- **RAM $\mathsf{Inc}$**: The architecture overview lists it as committed, but the RAM-specific page describes it as virtual (proven through sum-checks). Unclear which is correct.
- **RAM $\mathsf{wa}$**: With one memory op per cycle, there may be a single merged address polynomial rather than separate $\mathsf{ra}$ / $\mathsf{wa}$.

# TODO
- How Spartan R1CS glues the components (~20 constraints/cycle, PC updates, linking)
- RAM (Twist instance, memory layout, output verification)
- Virtual instructions (division decomposition, virtual registers 40-46)
- Inlines (custom instructions, SHA-256 benchmarks, virtual registers 47-63)
- The proof DAG: committed vs virtual polynomials, sum-check stages, where time is spent
- Interesting facts (e.g., d values per component)

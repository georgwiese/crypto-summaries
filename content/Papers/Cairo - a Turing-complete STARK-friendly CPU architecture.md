Goldberg, Papini, Riabzev ([Link](https://eprint.iacr.org/2021/1063.pdf))
# Design principles
- **Nondeterministic read-only memory**: Each memory cell can only have one value during the entire computation
	- One way to think of it is that the prover can choose the entire memory content, and the program asserts certain relationships among memory cells
	- This makes it trivial to pass prover hints: They simply become memory reads
	- -> There is no need to keep track of time stamps, since the value of a memory cell is independent of the time stamp
	- The verifier has a *partial memory function* and can verify it in linear time
	- The accessed memory cells need to form a contiguous region, and its size can be up to the number of cycles
- **Machine words = field elements**: They assume that the field has >63 bits (which suggests that the paper might be out of date, because M31 (used in stwo) is smaller...)
- **von Neumann architecture**: Code and data live in the same memory
	- -> The "PC lookup" becomes as memory read
	- Most instructions are encoded in a single word (using 63 bits). If the instruction has an immediate value, it is stored in a separate word
	- This enables techniques such as "bootloading", where the code is provided by the prover nondeterministically and verified to hash to a well-known value inside the zkVM
		- -> This makes the verification time sub-linear in the program size
	- It would also enable JITing the program: For example, instead of fetching an often-read value from memory, the program could modify its own code to specialize it to the current value
- **Monolithic CPU arithemtization**: Cairo's instruction set is very small and all core instructions are implemented in a single AIR
- **Builtins** (a.k.a precompiles) via **memory-mapped I/O** (see below)
# CPU architecture
## Registers
- **Program counter (pc)**: A memory pointer to the current instruction
- **Allocation pointer (ap)**: By convention points to the first memory cell that has not been used so far
- **Frame pointer (fp)**: Points to the beginning of the stack frame of the current function. When a function starts, `fp` is set to be the same as `ap`. When the function returns, the previous value of `fp` is restored.
# Example Program: Fibonacci
![[cairo_fibonacci.png]]
*Note: this example uses the allocation pointer to implement the loop. In section 8.1, the paper later states that loops are implement via tail recursion, suggesting that this is not the assembly code one would get when compiling from the Cairo programming language.*
## Instruction structure
The first word of each instruction is structured as follows:
![[cairo_instruction_structure.png]]
Note that fields that take multiple bits (e.g. `pc_update`) should only have 0 or 1 bit set (e.g. have values 0, 1, 2, or 4), so that each case can be selected by a degree-1 expression in the constraint system.

The semantics of the fields is as follows:
- `off_{dst,op0,op1}`: A signed integer, offset from the base pointer for the 3 operands
- `dst_reg`: If 0, the result is stored to an address relative to `ap`, otherwise to `fp`
- `op0_reg`: If 0, the first operand (`op0`) is relative to `ap`, otherwise to `fp`
- `op1_src`: Whether the second operand is relative to `op0`, `pc` (to read immediate values!), `fp`, or `ap`
- `res_logic`: Toggles between `res = op0`, `res = op0 + op1`, `res = op0 * op1`
- `pc_update`: Toggles between default PC update, absolute jump, relative jump, and conditional relative jump
- `opcode`: Handles `fp` and `ap` updates

If there is an immediate value (`op1_src == 1`), the immediate is stored at `pc + 1`.

All Cairo instructions can be implemented using this encoding.

## Calling convention
- `call` instruction:
	- Updates the PC similar to the `jmp` instruction
	- Stores the previous value of `fp` to `[ap]`
	- Stores the return address to `[ap + 1]`
	- Advances the `ap` by 2
- `ret` instruction: Restores the previous frame pointer, sets PC to the return address

By convention, the frame layout is as follows:
- **Arguments**: `[fp - 3], [fp - 4], ...`
- **Caller frame pointer**: `[fp - 2]`
- **Return address**: `[fp - 1]`
- **Local variables**: `[fp], [fp + 1], ...`
## Memory segments
Cairo has a concept of **memory segments**, such as:
- The program segment: Stores the bytecode
- Execution segment: Stores the execution stack
- User segments: General-purpose segments, dynamically allocated
- Builtin segments: Segments where builtins enforce additional constraints

During execution, memory addresses can be thought of as a tuple `(segment ID, address)`. When generating the proof, the prover will know the size of each segment and can allocate it into one linear memory. Note that because memory is read-only, the prover has complete freedom when allocating the segments, even overlapping segments would not pose a soundness issue.
## Builtins
Builtins (a.k.a precompiles) communicate with the CPU via **memory-mapped I/O**:
- For example, to range-check a value, it can be copied to a memory segment in which the range check builtin enforces all values to be in a certain range
- Builtins expose the start and end address of "their" segment as public values
- Also, the program needs to maintain the start and end pointer each builtin's segment, pass it to any function that needs to access the builtin and expose the final values as publics
- -> The verifier can check that both publics specify the same memory regions.
## Read/write memory
Cairo does not natively support read/write memory, but it can be emulated by:
- Having an append-only array (essentially a memory segment) of access logs, listing tuples of the form `(address, previous value, new value)`
- A builtin (I suppose) that stable-sorts the log by address (via a permutation check) and asserts that accesses are consistent

For details, refer to section 8.5.
## AIR constraints
TODO
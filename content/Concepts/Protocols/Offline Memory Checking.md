by Blum at all (1995)

[Explanation (YouTube)](https://youtu.be/dmVweFbJsxw?si=26si_BTIIcVzPb0m&t=4474)

The problem they tried to solve is to force an untrusted memory to prove it worked correctly, with very low space requirements of the verifier.

The verifier maintains two sets, which can be compressed and updated incrementally via permutation-invariant fingerprinting (see [[Permutation Check via Product Check]]):
- **Read Set**: The tuples $(address_i, value_i, count_i)$ returned by the prover
- **Write Set**: $(address_i, value_i, count_i)$ written to memory

Then, the verifier behaves as follows:
- At the beginning of the execution, (s)he writes for each address $a_i$ the tuple $(a_i, 0, 0)$ (or some other initial value)
- Each read returning $(a_i, v_i, c_i)$ is followed by a write of $(a_i, v_i, c_i + 1)$
- At the end of the execution, the verifier reads all addresses

If and only if the prover is honest, the read and write *multi* sets will be equal.
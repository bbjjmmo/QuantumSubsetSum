# Quantum Subset Sum via Grover's Algorithm

A Qiskit implementation of Grover's search algorithm applied to the subset sum problem, built from first principles using only single-qubit gates (with controls) and multi-controlled phase shifts.

## Overview

Given a list of integers `L` and a target sum, the algorithm finds all subsets of `L` that sum to the target. Subset selection is encoded in a boolean register (one qubit per element); a QFT-based adder computes the sum into an ancilla register in superposition; Grover's oracle phase-flips the matching states; and the diffuser amplifies them.

## Structure

The notebook (`grover_subset_sum.ipynb`) is organized as a pipeline of composable circuit-returning functions:

| Function | Description |
|---|---|
| `make_FrrT(n)` | QFT on `n` qubits: H + controlled-phase rotations (MSB first) + bit-reversal swaps |
| `make_adder(n, L)` | Draper QFT adder: FrrT → controlled phase shifts for each element → FrrT† |
| `make_U(n, L, target)` | Oracle: adder → X-flip zero bits of target → MCPhaseGate(π) → undo |
| `make_diffuser(n, L)` | Grover diffuser on the bool register: H → X → MCPhaseGate(π) → X → H |
| `grover_subset_sum(target, L)` | Full search: H on bool, repeated U + diffuser, returns high-probability bool states |

Each function returns a `QuantumCircuit`. Call `.to_gate()` on the output when composing into a larger circuit.

## Usage

```python
L = [1, 3, 4]
target = 4
for val, prob in grover_subset_sum(target, L):
    subset = [L[k] for k in range(len(L)) if (val >> k) & 1]
    print(f"bool={val:0{len(L)}b}  subset={subset}  prob={prob:.4f}")
```

Run all cells in order. The final cell prints the amplified boolean states and their subsets.

## Single vs Multiple Solutions

The number of Grover iterations used is:

```
num_iter = round(π/4 · √(2^n))
```

This formula is derived assuming **exactly one solution** (M = 1). In this case it is optimal and yields near-100% success probability.

When **multiple solutions exist** (M > 1), the optimal iteration count is:

```
k_opt = round(π / (4 · arcsin(√(M / 2^n))))
```

which is smaller. The M = 1 formula overshoots: applying too many iterations rotates the state past the solution and back toward uniform. For example, with `L = [1, 3, 4]` and `target = 4` there are M = 2 solutions ({1,3} and {4}); the optimal iteration count is k = 1, but the formula gives k = 2, which collapses the probability distribution back to near-uniform and returns all states above threshold rather than just the solutions.

Since M is unknown in advance, the current implementation uses the M = 1 approximation and is reliable when the target sum has a unique subset. Handling the multi-solution case requires quantum counting or an adaptive iteration strategy.

## Requirements

```
qiskit >= 2.4
pylatexenc
```

```bash
pip install qiskit pylatexenc
```

## Notes

- The sum register is an ancilla initialized to |0⟩ and uncomputed by the oracle; do not apply H to it
- Statevector simulation; cost scales as 2^(n + len(L)) where n = ceil(log2(sum(L) + 1))

## Author

Gillis

# Quantum Subset Sum via Grover's Algorithm

A Qiskit implementation of Grover's search algorithm applied to the subset sum problem, built from first principles using only single-qubit gates (with controls) and multi-controlled phase shifts.

## Overview

Given a list of integers `L` and a target sum, the algorithm finds all subsets of `L` that sum to the target. Subset selection is encoded in a boolean register (one qubit per element); a QFT-based adder computes the sum into an ancilla register in superposition; Grover's oracle phase-flips the matching states; and the diffuser amplifies them.

## Structure

The notebook (`grover_subset_sum_v2.ipynb`) is organized as a pipeline of composable circuit-returning functions:

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
- `num_iter = round(π/4 · √(2^n))` assumes one solution; cases with multiple solutions may overshoot and return near-uniform probabilities
- Statevector simulation; cost scales as 2^(n + len(L)) where n = ceil(log2(sum(L) + 1))

## Author

Gillis

# Quantum Subset Sum via Grover's Algorithm

A Qiskit implementation of Grover's search algorithm applied to the subset sum problem.

## Overview

Given a list of integers and a target sum, this notebook uses Grover's algorithm to find all subsets that sum to the target. The approach encodes the subset selection in a boolean register and uses a QFT-based adder to compute sums in superposition.

## Structure

- **FrrT gate** — custom QFT implementation (Hadamard + controlled phase rotations + bit-reversal swaps)
- **U_adder** — QFT-based adder: applies FrrT, adds controlled phase shifts for each element, applies FrrT†
- **U_oracle** — subset sum oracle: computes sum into ancilla register, phase-flips states matching `target_sum`, uncomputes
- **G** — full Grover operator: U_oracle followed by a diffuser on the boolean register
- **Output cell** — reads marginal probabilities over the boolean register, returns all states above threshold

## Usage

Set `input_list` and `target_sum` in cell 3, then run all cells in order. The final cell prints the high-probability boolean states after Grover's search.

```python
input_list = [1, 3, 4]
target_sum = 4
```

## Requirements

```
qiskit >= 2.4
qiskit-aer >= 0.17
pylatexenc
```

Install via:
```bash
pip install qiskit qiskit-aer pylatexenc
```

## Notes

- The sum register is ancilla and must start at |0⟩ — do not apply H to it before the search
- The optimal number of Grover iterations is `floor(π/4 · √(N/M))` where N = 2^(len(input_list)) and M = number of solutions; M is unknown in advance, so the current implementation uses an approximation
- When no solution exists, all boolean states remain at uniform probability and no high-probability state is reported
- Statevector simulation scales as 2^(register_length + len(input_list)); tested up to 15 qubits (input lists of 8 elements)

## Author

Gillis

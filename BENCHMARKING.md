# Benchmarking

Statevector simulation of `grover_subset_sum` over all lists of 1–4 elements (each in 1–8) and all valid targets up to 32. Measured on a single CPU core; no GPU acceleration.

## Results

| List length | Cases | Min (s) | Mean (s) | Max (s) | Total (s) |
|---|---|---|---|---|---|
| 1 | 8 | 0.005 | 0.008 | 0.011 | 0.06 |
| 2 | 100 | 0.008 | 0.019 | 0.080 | 1.9 |
| 3 | 680 | 0.011 | 0.032 | 0.274 | 21.6 |
| 4 | 3240 | 0.031 | 0.066 | 0.693 | 214.0 |

4028 cases total.

## Scaling

Each additional list element adds one bool qubit, doubling the statevector. It also increases the Grover iteration count by a factor of √2 (since `num_iter ~ √(2^n)`). Together, mean runtime roughly doubles per additional element.

The max within each length is higher than the mean because larger element values require more sum-register bits (n = ⌈log₂(Σ L + 1)⌉), pushing total qubit count up and amplifying cost.

## Qubit count

Total qubits = n + len(L), where n = ⌈log₂(sum(L) + 1)⌉.

| List length | Typical n | Total qubits |
|---|---|---|
| 1 | 4 | 5 |
| 2 | 5 | 7 |
| 3 | 5–6 | 8–9 |
| 4 | 6 | 10 |

Statevector simulation cost scales as 2^(total qubits), so each added qubit doubles memory and runtime.

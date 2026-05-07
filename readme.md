# Shor's Algorithm Implementation for YQuantum 2025 Hackathon

**Update:** This submission won first prize as Virtual Participants! 🥳

![image](https://github.com/user-attachments/assets/2e9f8236-2d6b-49e5-80b1-be2e2eead25b)

## Overview
This document describes an implementation of Shor's quantum algorithm for factoring semiprime integers, developed for the YQuantum 2025 Hackathon using the Quantum Rings SDK.

## Code
Repository: [Quantum-bits-YQuantum-2025](https://github.com/Ryukijano/Quantum-bits-YQuantum-2025)

Description: The Python implementation targets the QuantumRingsLib SDK (v0.9.0 during development) and the `scarlet_quantum_rings` backend simulator.

### Core logic
- Classical pre-processing (GCD check).
- Quantum period finding using Quantum Phase Estimation (QPE).
- Initialization of phase and target registers.
- Controlled modular exponentiation via repeated squaring.
- Inverse Quantum Fourier Transform (IQFT) on the phase register.
- Measurement of the phase register.
- Classical post-processing with continued fractions to find the period `r` and compute factors via GCD.

### Quantum arithmetic
The implementation includes custom-built quantum arithmetic circuits using basic gates (H, CX, CP, T, Tdg, SWAP) from QuantumRingsLib:
- QFT and IQFT circuits.
- CCX (Toffoli) decomposed into basic gates.
- CCCX (controlled-Toffoli) decomposed from CCX and CX.
- Multi-controlled X (`multi_controlled_X`) built from CCX/CCCX.
- Controlled modular adder for constants (`controlled_modular_adder_const`) using a QFT-based adder (mod 2^n) with explicit modular reduction (comparison via MSB flag, conditional add/subtract, and uncomputation).
- Controlled modular multiplier (`controlled_modular_multiplier`) using the accumulator method on the controlled modular adder.

## Results
Successfully factored integer: **N = 15** (L = 4 bits)

Prime factors: **3** and **5**

Parameters (N = 15):
- Base `a` = 7
- Shots = 2048

Qubits used (N = 15):
- Phase register (2n): 8 qubits
- Target register (n): 4 qubits
- Ancilla register (2n + 1): 9 qubits
- Total: 21 qubits

### Gate operations
A precise gate count is difficult to obtain automatically from the current setup. Circuit depth and gate count grow significantly with `n` (roughly O(n^3) or more for standard modular exponentiation). The implementation uses single-qubit gates (H, T, Tdg), two-qubit gates (CX, CP, SWAP), and multi-controlled gates (CCX, CCCX, MCP, MCT), all decomposed into basic gates.

### Execution time (N = 15)
The simulation on `scarlet_quantum_rings` completed successfully. Actual time depends on simulator performance and load, but was feasible for N = 15.

### Scaling attempts
The framework was configured for larger N values (e.g., N = 143, L = 8, requiring 41 qubits). Initial attempts encountered backend errors related to qubit limits or account permissions ("Either the user is not enabled or has access to fewer qubits than requested"). This prevented testing the full circuit execution for L = 8 or higher during this development phase. Further investigation with Quantum Rings support or account verification would be needed to run larger instances.

## Quantum Rings email
The email address used to register with Quantum Rings for this hackathon is: cbjp404@leeds.ac.uk

## Documentation / explanation
### Algorithm
Our implementation follows the standard textbook description of Shor's algorithm. It uses QPE to find the period `r` of `f(x) = a^x mod N`. Classical pre- and post-processing sandwich the quantum period-finding core. The period `r` is extracted from measured phases using continued fractions, which then allows classical calculation of factors via `gcd(a^(r/2) ± 1, N)`.

### Implementation details
- **Universality:** The code is parameterized by `L_to_run`, allowing selection of different N from `semiprimes.py`. Register sizes (2n phase, n target, 2n + 1 ancilla) are calculated dynamically based on `n = L_to_run`.
- **Modular exponentiation:** Implemented via repeated squaring. The circuit iterates through the 2n phase qubits. For each phase qubit `j`, a controlled modular multiplier applies `* (a^(2^j) mod N)` to the target register.
- **Controlled modular multiplier (CMM):** Uses the accumulator method. It iterates through the n qubits of the target register |y⟩. If the i-th qubit |y_i⟩ and the main control qubit are |1⟩, it adds `(k * 2^i) mod N` (where `k = a^(2^j) mod N`) to an n-qubit accumulator register (using ancillas). After iterating through all i, the result in the accumulator is controllably swapped into the target register, and the accumulator is cleaned via uncomputation (calling the adder's inverse).
- **Controlled modular adder:** Implemented as `controlled_modular_adder_const`. It uses a QFT-based adder for the initial `+ k_add mod 2^n` step. It then explicitly implements the modular reduction logic: subtract N (mod 2^n), check the MSB using a multi-controlled X onto a flag ancilla, conditionally add N back (mod 2^n) controlled by the flag, and finally uncompute the flag and the subtraction of N. This ensures the result is correctly computed modulo N. The inverse function is also implemented for uncomputation within the CMM.
- **Multi-controlled gates:** CCX is decomposed using H, T, Tdg, CX. CCCX and `multi_controlled_X` are decomposed using CCX and CX. Multi-controlled phase gates (MCP) are decomposed for up to two controls.

### Scalability
Theoretically, Shor's algorithm scales polynomially, a significant advantage over classical methods. The dominant cost is modular exponentiation, typically scaling around O(n^3) gates.

Practically, the number of qubits (~4n) and the large gate count make simulation challenging for large n. Our implementation requires 4n + 1 qubits plus any needed for multi-controlled gate decompositions.

Simulation time on classical hardware grows exponentially with the number of qubits for statevector simulation, although specialized simulators like Quantum Rings aim to push this limit higher. The `scarlet_quantum_rings` backend is designed for large qubit counts.

The primary bottleneck encountered was backend access/permissions for the required qubit counts for L >= 8.

### Insights, learnings, novelty
- Implementing quantum arithmetic from basic gates is feasible but complex, requiring careful attention to controls, modular reduction, and reversible uncomputation.
- The QFT adder provides an efficient core for addition, but the modular reduction logic adds significant complexity.
- Decomposing multi-controlled gates is necessary but increases circuit depth and width.
- Verifying complex, automatically generated quantum circuits remains a major challenge.
- The successful factorization of N = 15 validates the implemented logic for small scales. The framework is ready for larger N pending resolution of backend access issues or further optimization and verification.
- While standard techniques were primarily used, the successful integration and execution on the QuantumRingsLib platform for N = 15 demonstrates a complete implementation cycle.

Shor's Algorithm Implementation for YQuantum 2025 Hackathon
This document details the implementation of Shor's quantum algorithm for factoring large semiprime integers, developed for the YQuantum 2025 Hackathon using the Quantum Rings SDK.

1. Code
Repository: [](https://github.com/Ryukijano/Quantum-bits-YQuantum-2025)].

Description: The provided Python script implements Shor's algorithm targeting the QuantumRingsLib SDK (v0.9.0 was used during development) and the scarlet_quantum_rings backend simulator.

Core Logic: The implementation follows the standard Shor's algorithm procedure:

Classical pre-processing (GCD check).

Quantum period finding using Quantum Phase Estimation (QPE).

Initialization of phase and target registers.

Controlled Modular Exponentiation based on the repeated squaring method.

Inverse Quantum Fourier Transform (IQFT) on the phase register.

Measurement of the phase register.

Classical post-processing using the Continued Fractions Algorithm to determine the period r and calculate factors via GCD.

Quantum Arithmetic: The implementation includes custom-built quantum arithmetic circuits using basic gates (H, CX, CP, T, Tdg, Swap) provided by QuantumRingsLib:

QFT and IQFT circuits.

CCX (Toffoli) gate decomposed from basic gates.

CCCX (controlled-Toffoli) gate decomposed from CCX and CX gates.

Multi-controlled X gate (multi_controlled_X) decomposed using CCX/CCCX.

Controlled Modular Adder for constants (controlled_modular_adder_const) using a QFT-based adder (mod 2^n) combined with explicit modular reduction (mod N) logic (comparison via MSB flag, conditional addition/subtraction, and uncomputation steps).

Controlled Modular Multiplier (controlled_modular_multiplier) using the accumulator method, built upon the controlled modular adder.

2. Results
Successfully Factored Integer:

N = 15 (L=4 bits)

Prime Factors Found:

3 and 5

Parameters Used (N=15):

Base a = 7

Shots = 2048

Qubits Used (N=15):

Phase Register (2n): 8 qubits

Target Register (n): 4 qubits

Ancilla Register (2n+1): 9 qubits

Total: 21 qubits

Gate Operations:

A precise gate count is difficult to obtain automatically from the current setup.

The circuit depth and gate count grow significantly with n (roughly O(n^3) or more for standard modular exponentiation).

The implementation uses a combination of single-qubit gates (H, T, Tdg), two-qubit gates (CX, CP, SWAP), and multi-controlled gates (CCX, CCCX, MCP, MCT) which are decomposed into basic gates.

Execution Time (N=15):

The simulation on scarlet_quantum_rings completed successfully. (Actual time depends on the simulator's performance and load, but was feasible for N=15).

Scaling Attempts:

The framework was configured for larger N values (e.g., N=143, L=8, requiring 41 qubits).

Initial attempts encountered backend errors related to qubit limits or account permissions ("Either the user is not enabled or has access to fewer qubits than requested"). This prevented testing the full circuit execution for L=8 or higher during this development phase. Further investigation with Quantum Rings support or account verification would be needed to run larger instances.

3. Quantum Rings Email
The email address used to register with Quantum Rings for this hackathon is: cbjp404@leeds.ac.uk

4. Documentation / Explanation
Algorithm: Our implementation follows the standard textbook description of Shor's algorithm. It leverages QPE to find the period r of f(x) = a^x mod N. The key steps are classical pre/post-processing sandwiching the quantum period-finding core. The period r is extracted from measured phases using the Continued Fractions Algorithm, which then allows classical calculation of factors via gcd(a^(r/2) ± 1, N).

Implementation Details:

Universality: The code is parameterized by L_to_run, allowing selection of different N from semiprimes.py. Register sizes (2n phase, n target, 2n+1 ancilla) are calculated dynamically based on n=L_to_run.

Modular Exponentiation: Implemented via repeated squaring. The circuit iterates through the 2n phase qubits. For each phase qubit j, a Controlled Modular Multiplier (CMM) applies the operation * (a^(2^j) mod N) to the target register.

CMM: Uses the accumulator method. It iterates through the n qubits of the target register |y⟩. If the i-th qubit |y_i⟩ and the main control qubit are |1⟩, it adds (k * 2^i) mod N (where k = a^(2^j) mod N) to an n-qubit accumulator register (using ancillas). This addition is performed by the controlled_modular_adder_const function. After iterating through all i, the result in the accumulator is controllably swapped into the target register, and the accumulator is cleaned via uncomputation (calling the adder's inverse).

Controlled Modular Adder: Implemented as controlled_modular_adder_const. It uses a QFT-based adder for the initial + k_add mod 2^n step. It then explicitly implements the modular reduction logic: subtract N (mod 2^n), check the MSB using a multi-controlled X onto a flag ancilla, conditionally add N back (mod 2^n) controlled by the flag, and finally uncompute the flag and the subtraction of N. This ensures the result is correctly computed modulo N. The inverse function is also implemented for uncomputation within the CMM.

Multi-Controlled Gates: CCX is decomposed using H, T, Tdg, CX. CCCX and multi_controlled_X are decomposed using CCX and CX. Multi-controlled phase gates (mcp) are decomposed for up to 2 controls.

Scalability:

Theoretically, Shor's algorithm scales polynomially, a significant advantage over classical methods. The dominant cost is the modular exponentiation, typically scaling around O(n^3) gates.

Practically, the number of qubits (~4n) and the immense gate count make simulation challenging for large n. Our implementation requires 4n+1 qubits plus any needed for multi-controlled gate decompositions.

Simulation time on classical hardware grows exponentially with the number of qubits if using statevector simulation, although specialized simulators like Quantum Rings aim to push this limit higher. The scarlet_quantum_rings backend is designed for large qubit counts.

The primary bottleneck encountered was backend access/permissions for the required qubit counts for L >= 8.

Insights/Learnings/Novelty:

Implementing quantum arithmetic from basic gates is feasible but highly complex, requiring careful attention to controls, modular reduction, and particularly reversible uncomputation.

The QFT adder provides an efficient core for addition, but the modular reduction logic adds significant complexity.

Decomposing multi-controlled gates is necessary but increases circuit depth/width.

Verifying complex, automatically generated quantum circuits remains a major challenge.

The successful factorization of N=15 validates the implemented logic for small scales. The framework is ready for larger N pending resolution of backend access issues or further optimization/verification.

While standard techniques were primarily used, the successful integration and execution on the QuantumRingsLib platform for N=15 demonstrates a complete implementation cycle.

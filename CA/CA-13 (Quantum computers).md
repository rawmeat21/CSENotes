# Quantum Computing — Exam Notes

> PYQ coverage: 2025 Q37, Q38 | Cross-referenced throughout

---

## 1. Why Quantum Computing? — Moore's Law & Its Limits

**Moore's Law** states that the number of transistors in a dense integrated circuit doubles approximately every two years. This trend has held since Gordon Moore observed it in 1965.

The physical problem: as transistors shrink to atomic/molecular scale, classical mechanics breaks down. At that scale, particles obey **quantum mechanics**, not Newtonian physics. This is both the challenge and the opportunity — quantum effects can be exploited for computation.

**Richard Feynman** (1959, "There's Plenty of Room at the Bottom") noted that circuits of just a few atoms obey quantum mechanical laws, opening completely new opportunities for design.

---

## 2. What Is a Quantum Computer?

**Classical Computer:** Uses voltages flowing through circuits and gates. All behaviour is governed by classical mechanics. Bits are either 0 or 1 — definite, deterministic states.

**Quantum Computer:** Uses the laws of quantum mechanics to perform massively parallel computation through **superposition**, **entanglement**, and **decoherence**.

---

## 3. Classical vs. Quantum Mechanics

| Property | Classical Mechanics | Quantum Mechanics |
|---|---|---|
| Deals with | Macroscopic particles | Microscopic particles (electrons, photons, atoms) |
| Foundation | Newton's laws + Maxwell's EM theory | Schrödinger equation |
| Energy | Emitted/absorbed continuously | Only discrete values (Planck's postulation — origin of the word "quantum") |
| State | Exactly defined by position + velocity | Cannot be exactly specified (Heisenberg uncertainty + wave-particle duality) |
| Prediction | Future state certain | Gives probabilities of finding particles at locations |

---

## 4. Quantum Mechanics — The Mathematical Foundation

### 4.1 The Schrödinger Equation

A quantum state $|\psi\rangle$ evolves in time according to:

$$\frac{d|\psi\rangle}{dt} = -\frac{i}{\hbar}\hat{H}(t)|\psi\rangle$$

where $\hat{H}$ is the **Hamiltonian** operator (total energy of the system) and $\hbar$ is the reduced Planck constant.

This is a **linear** differential equation. Because it is linear:
- Its solutions obey the **superposition principle**: linear combinations of solutions are also solutions.
- This is the mathematical origin of quantum superposition.

### 4.2 Unitary Time Evolution

The time evolution of a quantum state is described by a **unitary transformation**:

$$|\psi\rangle \rightarrow \hat{U}|\psi\rangle$$

$$\frac{d\hat{U}(t)}{dt} = -\frac{i}{\hbar}\hat{H}(t)\hat{U}(t)$$

Since all unitary operators are invertible, every quantum computation can always be **reversed** (uncomputed).

### 4.3 The Three Phases of Quantum Computation

1. **Preparation:** The initial state $|\Psi(0)\rangle$ is prepared at $t_0 = 0$.
2. **State Evolution:** The state is evolved by a sequence of unitary operations $U(t_1, t_0),\ U(t_2, t_1),\ \ldots$
3. **Measurement:** The final state is measured. The probability of observing outcome $|\Phi\rangle$ is:
$$P(\Phi) = |\langle \Phi | \Psi(n) \rangle|^2$$
Measurement is **projective** — it collapses the wavefunction.

---

## 5. Qubit — The Quantum Bit

A **qubit** is the smallest unit of information in a quantum computer. It represents the state $|\psi\rangle$ of the wavefunction in the Schrödinger equation.

### 5.1 Physical Implementations

| Physical System | $|0\rangle$ | $|1\rangle$ | Used by |
|---|---|---|---|
| Nuclear spin (NMR) | Spin up $\uparrow$ | Spin down $\downarrow$ | Early QCs |
| Photons in cavity | 0 photons | 1 photon | Chinese 76-qubit system |
| Atomic energy levels | Ground state | Excited state | Trapped ions (IonQ, Honeywell) |
| Superconducting loops | Right current | Left current | IBM, Google |
| Silicon quantum dots | Electron spin state 0 | Electron spin state 1 | Intel |

### 5.2 General Qubit State — Superposition

Since the Schrödinger equation is linear, if $|0\rangle$ and $|1\rangle$ are valid states, their linear combination is also valid. The **general state** of a qubit is:

$$|\psi\rangle = \alpha_0|0\rangle + \alpha_1|1\rangle$$

where $\alpha_0$ and $\alpha_1$ are **complex numbers** called **probability amplitudes**.

- $|\alpha_0|^2$ = probability of measuring $|0\rangle$ (the "off" state)
- $|\alpha_1|^2$ = probability of measuring $|1\rangle$ (the "on" state)
- **Normalization condition:** $|\alpha_0|^2 + |\alpha_1|^2 = 1$

### 5.3 The Bloch Sphere

A qubit state can be visualized as a point on the surface of a unit sphere called the **Bloch Sphere**:

$$|\Psi\rangle = e^{i\gamma}\left(\cos\frac{\theta}{2}|0\rangle + e^{i\varphi}\sin\frac{\theta}{2}|1\rangle\right)$$

- North pole ($\hat{z}$) = $|0\rangle$
- South pole ($-\hat{z}$) = $|1\rangle$
- $\theta$ = polar angle, $\varphi$ = azimuthal angle

### 5.4 Classical Bit vs. Quantum Bit

| Property | Classical Bit | Quantum Bit |
|---|---|---|
| States | 0 or 1 (definite) | $|0\rangle$, $|1\rangle$, or any superposition |
| Measurement | Complete, non-disturbing | Partial, probabilistic, changes the state |
| Copying | Can be copied freely | **Cannot be copied** (No-Cloning Theorem) |
| Erasure | Can be erased | **Cannot be erased** (No-Deletion Theorem) |

### 5.5 Exponential Power of Qubits

- A classical $n$-bit computer operates on a single $n$-bit number at a time.
- A quantum $n$-qubit computer operates in a Hilbert space of $2^n$ dimensions simultaneously.
- **To double a classical computer's power:** 32 bits → 64 bits (double the hardware).
- **To double a quantum computer's power:** 32 qubits → 33 qubits (add just one qubit).
- A 64-qubit computer operates in a space of $2^{64} \approx 1.6 \times 10^{19}$ dimensions.
- One operation on $2^n$ numbers encoded by $n$ qubits takes **1 step** vs $2^n$ steps classically.

---

## 6. Superposition

> **PYQ 2025 Q38** asks about decoherence; understanding superposition first is essential.

Every quantum state can be represented as a sum of two or more other distinct states. Mathematically, because the Schrödinger equation is linear, any linear combination of solutions is also a solution.

A single qubit in superposition:

$$|\psi\rangle = \alpha_0|0\rangle + \alpha_1|1\rangle \quad \text{where } |\alpha_0|^2 + |\alpha_1|^2 = 1$$

The qubit is simultaneously in both $|0\rangle$ and $|1\rangle$. When **measured**, it collapses to one of them with probability $|\alpha_0|^2$ or $|\alpha_1|^2$.

**Example — 3-qubit register:** An equally-weighted superposition of all 8 possible states:

$$|\psi\rangle = \frac{1}{\sqrt{8}}|000\rangle + \frac{1}{\sqrt{8}}|001\rangle + \cdots + \frac{1}{\sqrt{8}}|111\rangle$$

---

## 7. Entanglement

> **PYQ 2025 Q38** — Entanglement is listed as a distinct concept from decoherence.

**Entanglement** occurs when two or more particles are generated or interact such that the quantum state of each particle **cannot be described independently** of the others, even when separated by large distances.

Properties:
- An entangled pair forms a single quantum system in superposition of equally possible states.
- The entangled state contains **no information about individual particles**, only their correlation (they are in opposite states).
- If the state of one particle is changed, the other **instantly** adjusts to be consistent with quantum mechanics.
- If one particle is measured, the other **automatically collapses**.
- Einstein called this **"spooky actions at a distance"** and was deeply uncomfortable with it.
- Entanglement is **a primary feature of quantum mechanics that has no classical counterpart**.

### 7.1 Tensor Product — Mathematical Description

If two independent qubits are in states $\alpha|0\rangle + \beta|1\rangle$ and $\alpha'|0\rangle + \beta'|1\rangle$, the combined system is their **tensor product**:

$$(\alpha|0\rangle + \beta|1\rangle)(\alpha'|0\rangle + \beta'|1\rangle) = \alpha\alpha'|00\rangle + \alpha\beta'|01\rangle + \beta\alpha'|10\rangle + \beta\beta'|11\rangle$$

**Independent state** (can be factored):
$$\frac{1}{2}|00\rangle + \frac{1}{2}|01\rangle - \frac{1}{2}|10\rangle - \frac{1}{2}|11\rangle = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle) \otimes \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$$

**Entangled state** (cannot be factored into individual qubit states):
$$\frac{1}{\sqrt{2}}|00\rangle + \frac{1}{\sqrt{2}}|11\rangle$$
This is a **Bell state** — there is no assignment of individual qubit states that reproduces it.

---

## 8. Decoherence

> **PYQ 2025 Q38 directly asks:** "The loss of information from a quantum system into the environment is called (a) Superposition (b) Entanglement **(c) Decoherence** (d) none of the above"
>
> **Answer: (c) Decoherence**

**Quantum decoherence** is the loss of superposition due to the spontaneous interaction between a quantum system and its environment.

Decoherence can be understood as the **loss of information from a quantum system into the environment**. When the environment interacts with the qubits, the delicate phase relationships that define superposition are destroyed, and the qubit effectively behaves like a classical bit.

- Superposition and entanglement are extremely fragile states.
- Any interaction with the environment (heat, vibration, electromagnetic noise) can cause decoherence.
- **Preventing decoherence is the single biggest engineering challenge in building quantum computers.**
- This is why superconducting quantum computers operate at $\approx 0.02$ K (millikelvin), colder than outer space.

---

## 9. Measurement

- If a quantum system is perfectly isolated, it maintains coherence indefinitely, but it becomes impossible to interact with or extract results from.
- A quantum **measurement is itself a decoherence process** — the act of measuring disturbs the system.
- When $|\psi\rangle$ is measured, the wavefunction **collapses** to either $|0\rangle$ or $|1\rangle$ with probabilities $|\alpha_0|^2$ and $|\alpha_1|^2$.
- A measurement **never** produces a superposition state as output — you always get a definite classical result.

**Example:** $|\psi\rangle = 0.316|00\rangle + 0.447|01\rangle + 0.548|10\rangle + 0.632|11\rangle$

Probability that the rightmost bit reads 0:
$$|0.316|^2 + |0.548|^2 = 0.1 + 0.3 = 0.4$$

---

## 10. Quantum Gates

> **PYQ 2025 Q37:** "In Quantum Logic Circuits, operations are (a) **defined by Linear Algebra over Hilbert Space and can be represented by unitary matrices with complex elements** (b) defined by Boolean Algebra"
>
> **Answer: (a)**

A **quantum gate** (quantum logic gate) is a basic quantum circuit element operating on qubits. Quantum gates are the building blocks of quantum circuits, analogous to classical logic gates.

**Key properties of quantum gates:**
- Every gate operation $U$ must be **unitary**: $UU^\dagger = I$ (preserves normalization)
- A gate acting on $n$ qubits is represented by a $2^n \times 2^n$ **unitary matrix with complex elements**
- Operations are defined by **linear algebra over Hilbert space**
- The number of input and output qubits must be equal
- Unlike many classical gates, quantum gates are **reversible** (because unitary operators are invertible)

### 10.1 Unitary Matrix

A complex square matrix $U$ is **unitary** if:
$$U^*U = UU^* = I$$

Unitary transformations are linear transformations that preserve the norm (length) of vectors. In 2D, they correspond to rotations and reflections on the unit circle.

General $2\times2$ unitary matrix form:
$$U = \begin{bmatrix} a & b \\ -e^{i\varphi}b^* & e^{i\varphi}a^* \end{bmatrix}, \quad |a|^2 + |b|^2 = 1$$

### 10.2 Single-Qubit Gates

**Pauli-X Gate (quantum NOT):**
$$X = \begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix}$$
$$|0\rangle \rightarrow |1\rangle, \quad |1\rangle \rightarrow |0\rangle$$
When acting on pure basis states, this is identical to a classical NOT gate.

Verification:
$$X\begin{bmatrix}1\\0\end{bmatrix} = \begin{bmatrix}0&1\\1&0\end{bmatrix}\begin{bmatrix}1\\0\end{bmatrix} = \begin{bmatrix}0\\1\end{bmatrix} = |1\rangle \checkmark$$

**Pauli-Y Gate:**
$$Y = \begin{bmatrix} 0 & -i \\ i & 0 \end{bmatrix}$$
$$|0\rangle \rightarrow i|1\rangle, \quad |1\rangle \rightarrow -i|0\rangle$$
Has no classical equivalent.

**Pauli-Z Gate:**
$$Z = \begin{bmatrix} 1 & 0 \\ 0 & -1 \end{bmatrix}$$
Flips the phase of $|1\rangle$ but leaves $|0\rangle$ unchanged.

**Phase Gate (S):**
$$S = \begin{bmatrix} 1 & 0 \\ 0 & i \end{bmatrix}, \quad S^2 = Z$$

**$\pi/8$ Gate (T gate):**
$$T = \begin{bmatrix} 1 & 0 \\ 0 & e^{i\pi/4} \end{bmatrix}$$

### 10.3 The Hadamard Gate (H Gate)

The Hadamard gate is **one of the most important gates in quantum computing**.

$$H = \frac{1}{\sqrt{2}}\begin{bmatrix} 1 & 1 \\ 1 & -1 \end{bmatrix}$$

**Action:**
$$|0\rangle \rightarrow \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle), \quad |1\rangle \rightarrow \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle)$$

**Inverse actions (H applied again reverses):**
$$\frac{|0\rangle + |1\rangle}{\sqrt{2}} \rightarrow |0\rangle, \quad \frac{|0\rangle - |1\rangle}{\sqrt{2}} \rightarrow |1\rangle$$

**Key properties:**
- $H$ is its own inverse: $H^{-1} = H$ and $H^2 = I$
- Applying $H$ to a deterministic state $|0\rangle$ or $|1\rangle$ produces a **truly random 50/50 superposition**
- Applying $H$ again to that random superposition produces a **deterministic outcome** — applying a randomizing operation to a random state produces a deterministic outcome
- Has **no classical equivalent**

### 10.4 Two-Qubit Gate: CNOT (Controlled NOT)

$$CNOT = \begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 0 & 1 \\ 0 & 0 & 1 & 0 \end{bmatrix}$$

**Action:**
- Control qubit = 0 → target qubit unchanged
- Control qubit = 1 → target qubit is flipped (NOT applied)

$$|00\rangle \rightarrow |00\rangle, \quad |01\rangle \rightarrow |01\rangle, \quad |10\rangle \rightarrow |11\rangle, \quad |11\rangle \rightarrow |10\rangle$$

CNOT is equivalent to the classical **XOR gate** (output true iff exactly one input is true).

### 10.5 Summary Table of Quantum vs Classical Gates

| Quantum Operator | Matrix | Classical Equivalent |
|---|---|---|
| Pauli-X (X) | $\begin{bmatrix}0&1\\1&0\end{bmatrix}$ | NOT |
| Pauli-Y (Y) | $\begin{bmatrix}0&-i\\i&0\end{bmatrix}$ | None |
| Pauli-Z (Z) | $\begin{bmatrix}1&0\\0&-1\end{bmatrix}$ | None |
| Hadamard (H) | $\frac{1}{\sqrt{2}}\begin{bmatrix}1&1\\1&-1\end{bmatrix}$ | None |
| Phase (S) | $\begin{bmatrix}1&0\\0&i\end{bmatrix}$ | None |
| $\pi/8$ (T) | $\begin{bmatrix}1&0\\0&e^{i\pi/4}\end{bmatrix}$ | None |
| CNOT (CX) | $4\times4$ matrix above | XOR |
| Toffoli (CCX) | $8\times8$ matrix | AND (reversible) |

Classical gates: AND, OR, NAND, NOR, NOT, XOR — operations defined by **Boolean Algebra**.
Quantum gates: defined by **linear algebra over Hilbert space**, represented by **unitary matrices with complex elements**.

---

## 11. Quantum Circuit

A **quantum circuit** is a model for quantum computation in which a computation is a sequence of quantum gates applied to an $n$-qubit register connected by "wires."

**Key difference from classical circuits:** The same number of wires (qubits) passes throughout the entire circuit — no wire is added or removed. This is because quantum gates are unitary (number-preserving).

### 11.1 Classical Logic Circuits vs. Quantum Logic Circuits

| Property | Classical Logic Circuit | Quantum Logic Circuit |
|---|---|---|
| Governed by | Classical physics (implicitly) | Quantum mechanics (explicitly) |
| Signal states | Simple bit vectors, e.g. 01010111 | Superpositions of qubit vectors with complex coefficients |
| Operations defined by | Boolean Algebra | Linear algebra over Hilbert Space, represented by unitary matrices with complex elements |
| Copying/measuring signals | No restrictions | Severe restrictions (no-cloning, no-deletion) |
| Universal gate sets | Small, well-defined sets: $\{$NAND$\}$, $\{$AND, OR, NOT$\}$ | Many universal sets; best types not obvious |
| Technology | Fast, scalable, macroscopic (CMOS) | Slow, fragile, microscopic (NMR, superconductors) |

The general state of an $n$-qubit system:
$$|\Psi\rangle = \sum_{i=0}^{2^n - 1} c_i |i_{n-1} i_{n-2} \ldots i_0\rangle$$

### 11.2 Bell State — Generating Entanglement with a Circuit

A Bell state (maximally entangled 2-qubit state) is generated by applying a Hadamard gate followed by a CNOT gate to two qubits both initialized to $|0\rangle$:

**Step 1:** Apply $H$ to the first qubit:
$$H|0\rangle_1 \otimes |0\rangle_2 = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle) \otimes |0\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |10\rangle)$$

**Step 2:** Apply CNOT (first qubit = control, second = target):
$$\frac{1}{\sqrt{2}}(|00\rangle + |10\rangle) \xrightarrow{CNOT} \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$$

The result $\frac{|00\rangle + |11\rangle}{\sqrt{2}}$ is an **entangled Bell state** — it cannot be factored into individual qubit states.

---

## 12. Quantum Algorithms — Why Faster?

A quantum algorithm may be faster than any classical algorithm because of three advantages:

1. **Quantum Parallelism:** By using superposition, a quantum computer evaluates a function on all $2^n$ possible inputs simultaneously in a single step.
2. **Dimension of Hilbert Space:** The state space of an $n$-qubit system is $2^n$-dimensional, exponentially larger than the corresponding $n$-bit classical system.
3. **Entanglement Capability:** Qubits can be entangled, creating non-classical correlations that enable certain computations that have no efficient classical counterpart.

### 12.1 Algorithm Complexity Comparison

| Algorithm Type | Classical Steps | Quantum Steps |
|---|---|---|
| Fourier Transform (e.g. Shor's) | $N\log N = n \cdot 2^n$ | $\log^2(N) = n^2$ |
| Search (e.g. Grover's) | $N$ | $\sqrt{N}$ |
| Quantum Simulation | $c^N$ bits | $kn$ qubits |

### 12.2 Shor's Algorithm (Factoring)

Peter Shor (1994) developed the first quantum algorithm that demonstrates an **exponential speedup** over the best known classical algorithm for integer factoring.

**Problem:** Given large integer $N$, find its prime factors.

**Classical:** Best known algorithm (general number field sieve) takes $O(\exp(n^{1/3}))$ time. For a 1024-digit number on a classical THz computer: ~150,000 years.

**Shor's quantum algorithm:** Takes $O(n^3 \log n)$ time. On a quantum THz computer: under 1 second.

**How Shor's algorithm works:**
1. Select $x$ coprime to $N$
2. Use the Quantum Fourier Transform (QFT) to find the period of $f(s) = x^s \mod N$
3. Use the order (period) of $x$ to compute possible factors of $N$ using the number theory theorem: if $a \neq \pm b \pmod{N}$ but $a^2 \equiv b^2 \pmod{N}$, then $\gcd(a+b, N)$ is a factor of $N$
4. Check if the factors are correct; if not, rerun

**Implication for cryptography:** Public key cryptography (RSA) relies on the computational infeasibility of factoring large numbers. A quantum computer with ~8,000 logical qubits could crack RSA-2048.

### 12.3 Grover's Algorithm (Search)

Lov Grover (1996) developed a quantum algorithm that finds a marked entry in an **unsorted database** of $N$ entries in $O(\sqrt{N})$ steps, compared to $O(N)$ classically — a **quadratic speedup**.

Applications: fast searching of big data, inverting complex functions, pattern recognition.

### 12.4 Quantum Fourier Transform (QFT)

The QFT is the quantum analogue of the Discrete Fourier Transform, central to Shor's algorithm.

$$U_{QFT} = \frac{1}{\sqrt{N}} \sum_{y=0}^{N-1} \sum_{x=0}^{N-1} \omega_N^{xy} |y\rangle\langle x|$$

where $\omega_N = e^{2\pi i/N}$.

The inverse:
$$U_{QFT}^\dagger = \frac{1}{\sqrt{N}} \sum_{y=0}^{N-1} \sum_{x=0}^{N-1} \omega_N^{-xy} |x\rangle\langle y|$$

---

## 13. Design and Implementation of Quantum Computers

### 13.1 Architecture

A quantum computer consists of:
- **Classical computer** → sends programs (as gate commands) to a **quantum computer controller**
- **Quantum computer controller** → sends gate commands to and reads measurement results from the **quantum register** (the qubits)

Job flow: Submit job → Job Queue → Translate quantum circuit to microwave pulses → Execute on quantum computer → Return measurement results.

### 13.2 Requirements for a Quantum Computer

**Core requirements:**
- Qubit implementation (physical two-state system)
- Control of unitary evolution (applying gates)
- Initial state preparation (initialize qubits to $|0\rangle$)
- Measurement of the final state

**Engineering requirements:**
- System must be almost completely isolated from the environment (prevents decoherence)
- Coherent quantum state must be preserved throughout computation
- Decoherence times must be longer than algorithm execution time
- Ability to perform operations on several qubits in parallel
- Physical system with two uniquely addressable states
- A universal set of quantum gates
- Ability to entangle two qubits

### 13.3 Leading Hardware Technologies

**Superconducting Circuits** (IBM, Google):
- Qubit = direction of supercurrent in a loop (right or left)
- Connected with wires, fast gates, printable/VLSI-compatible
- Challenges: very short coherence time ($\sim 10^{-6}$ s), requires 0.05K cryogenics, all qubits different, not reconfigurable
- IBM 53-qubit and Google 72-qubit machines use this technology

**Trapped Atomic Ions** (Honeywell, IonQ):
- Qubit = internal energy states of individual ions held in place by electromagnetic fields
- Very long coherence time ($\gg 1$ sec), all qubits identical, connections reconfigurable
- Challenges: requires lasers, optics, high vacuum, 4K cryogenics

**Other approaches:** NV-Diamond (nitrogen vacancies in diamond), semiconductor quantum dots, neutral atoms in optical lattices, photonic qubits

### 13.4 Quantum Volume

**Quantum Volume $V_Q$** is a metric that measures both the number of qubits and the quality of those qubits (error rates), capturing the overall capability of a quantum computer.

$$\tilde{V}_Q = \min[N, d(N)]^2$$

where $N$ is the number of qubits and $d(N)$ is the circuit depth achievable. IBM's definition:
$$\log_2 V_Q = \underset{n \leq N}{\arg\max}\ \{\min[n, d(n)]\}$$

### 13.5 NISQ Era (Noisy Intermediate-Scale Quantum)

**NISQ** — coined by John Preskill (Caltech, 2018).

Current quantum computers:
- Have 50–100 qubits ("intermediate scale" — not too few, not enough for full fault-tolerance)
- Are "noisy" because there are not enough qubits to spare for quantum error correction — the imperfect physical qubits must be used directly
- Can perform tasks classical computers cannot, but are not yet fault-tolerant

Roadmap (Google strategy):
1. **Quantum Supremacy** (~50–100 qubits, current era)
2. **Near-term applications** — looking for practical NISQ-era algorithms
3. **Error correction** — many physical qubits per logical qubit
4. **Full fault-tolerant quantum computer** — $10^6$+ qubits

---

## 14. Applications of Quantum Computers

1. **Quantum Factoring** (Shor, 1994) — exponential speedup over classical; breaks RSA
2. **Quantum Search** (Grover, 1996) — quadratic speedup; unsorted database search
3. **Quantum Simulation** — simulating quantum systems (chemistry, materials). $N$ quantum systems classically require $2^N$ coupled equations; on a quantum simulator they require only $kN$ qubits
4. **Quantum Optimization** — minimizing complex nonlinear functions by sampling the full space through superposition; relevant to logistics, VLSI design, finance
5. **Quantum Networks / Cryptography** — Quantum Key Distribution (QKD), certifiable random number generation, quantum repeaters

### 14.1 Quantum Cryptography (QKD)

Alice sends random qubits to Bob choosing one of two bases: $\{|0\rangle, |1\rangle\}$ or $\{|0\rangle \pm |1\rangle\}$. Bob measures in a randomly chosen basis. If an eavesdropper Eve intercepts the qubits, she disturbs them (she doesn't know the basis). Alice and Bob then compare bases publicly and discard mismatched bits — the remaining bits form a shared secret key. Any eavesdropping is detectable from the disturbed statistics.

### 14.2 Variational Quantum Eigensolver (VQE)

VQE is a **quantum-classical hybrid algorithm** that finds eigenvalues of a Hamiltonian $H$ (e.g., the energy of a molecular system).

**Procedure:**
1. Map the physical Hamiltonian to a qubit Hamiltonian
2. Prepare a trial wavefunction $|\psi\rangle$ (the "ansatz") using a quantum circuit (Hadamard gates + $R_y$, $R_z$ rotations)
3. Measure the expectation value $\langle \psi | H | \psi \rangle$ on the quantum computer
4. Pass this value to a classical non-linear optimizer
5. The optimizer varies the ansatz parameters and sends back new gate angles
6. Repeat until $\langle H \rangle$ converges to the minimum (ground state energy)

VQE has been demonstrated on $H_2$, LiH, $BeH_2$ and $H_{12}$ molecules.

---

## 15. Problems with Quantum Computers

- **Decoherence** — quantum state collapses due to environmental interaction; decoherence time must exceed algorithm running time
- **Error correction** — requires many additional physical qubits for each logical qubit
- **Physical implementation** — microscopic technologies are slow, fragile, and not yet scalable
- **Algorithm development** — limited set of problems with known quantum speedup
- **Entangled state generation** — engineering reliable entanglement at scale is very hard
- **Post-quantum cryptography** — current encryption (RSA) is vulnerable; transition to quantum-safe algorithms is urgent

---

## 16. PYQ Direct Coverage

### 2025 Q37
**"In Quantum Logic Circuits, operations are (a) defined by Linear Algebra over Hilbert Space and can be represented by unitary matrices with complex elements (b) defined by Boolean Algebra"**

**Answer: (a)**

Classical circuits use Boolean Algebra (AND, OR, NOT on bits). Quantum circuits use linear algebra over Hilbert space — the state is a superposition vector in a $2^n$-dimensional complex Hilbert space, and gate operations are unitary matrices with complex elements. Each quantum gate $U$ satisfies $UU^\dagger = I$.

### 2025 Q38
**"The loss of information from a quantum system into the environment is called (a) Superposition (b) Entanglement (c) Decoherence (d) none of the above"**

**Answer: (c) Decoherence**

- **Superposition:** A qubit existing in a linear combination of $|0\rangle$ and $|1\rangle$ simultaneously — this is not a loss of information.
- **Entanglement:** A non-separable correlation between two or more qubits — this is not information loss.
- **Decoherence:** The process by which a quantum system loses its quantum properties (superposition, entanglement) through interaction with the environment. The quantum information leaks into the environment, destroying the coherence of the state. This is the correct definition.

---

## 17. Brief Historical Timeline

| Year | Event |
|---|---|
| 1925 | Schrödinger equation proposed |
| 1935 | EPR paradox — thought experiment on entanglement |
| 1981 | Feynman proposes using quantum phenomena for computation |
| 1984 | Quantum cryptography (IBM) |
| 1985 | David Deutsch describes first universal quantum computer |
| 1994 | Shor's prime factorization algorithm sparks interest in QC |
| 1995 | Schumacher proposes "qubit" as a term |
| 1996 | Grover's database search algorithm |
| 1998 | First 2-qubit quantum computers (NMR-based, Oxford/IBM/MIT/Stanford) |
| 2000 | 7-qubit NMR quantum computer (Los Alamos) |
| 2001 | IBM factors 15 = 3 × 5 on 7-qubit quantum computer |
| 2010 | D-Wave announces first commercial QC |
| 2016 | IBM makes 5-qubit QC available via cloud |
| 2017 | IBM: 50-qubit processor; Microsoft reveals Q# |
| 2019 | Google claims quantum supremacy (54 qubits, 200 sec vs 10,000 classical years); IBM: 53 qubits |
| 2020 | Chinese researchers: 76-qubit photonic system at 100 trillion× classical speed |

---

## 18. Suggested Further Study (Look These Up)

- **Quantum error correction codes** (Shor code, stabilizer codes) — the PDF does not cover these in depth but they are essential for fault-tolerant QC
- **Grover's algorithm circuit** — the PDF mentions it but doesn't show the full circuit; look up the oracle + diffusion operator construction
- **Post-quantum cryptography standards** — NIST has finalized lattice-based standards (CRYSTALS-Kyber, CRYSTALS-Dilithium) as of 2024; worth knowing for context
# QC-Project
Ground State Preparation Project for Quantum Computing Course

# Aim

Examine the ground state manifold of a one-dimensional Hamiltonian, describing the quantum spins with a nearest-neighbour magnetic coupling.

**Hamiltonian**:
\
$H=\sum_{i=1}^{L} (X_i \otimes X_{i+1} + Y_i \otimes Y_{i+1} + J_zZ_i \otimes Z_{i+1})$

where $X_i, Y_i$ and $Z_i$ are the Pauli matrices acting on site $i$. This is a simplified model of a magnetic insulator. The important coefficient is $J_z$, the anisotropy term. It determines the strength of the \textit{Z-Z}
interaction, compared to the other two interaction terms (namely \textit{X-X} and \textit{Y-Y}). This model has two quantum phase transitions as $J_z$ varies, one at $J_z = 1$ and one at $J_z = -1$. These correspond to different types of magnetic configurarions. The quantum algorithm implemented will aim to find the ground state of this Hamiltonian across the interval $J_z \in [-3,3]$, and from these ground states identify the location of the phase transitions. The phase transition at $J_z = -1$ is Berezinskii–Kosterlitz–Thouless transition, requiring larger systems with more thorough entanglement to be resolved.

Work included testing different Variational Quantum Eigensolver Algorithms to best find the ground state whilst exploring the phase space through a antiferromagnetic structure factor, S($\pi$). Testing the quality of the eignesolver and it's ability to find the groundstate was achieved by comparing simulated runs (using GenericBackend) and real hardware runs on IBM Fez. As runtime was constricted on IBM Fez, a Gaussian Process Regressor was trained on a 72-configuration VQE sweep (with different ansatz and optimizers) to identify the best performing settings (COBYLA and RealAmplitudes with 12 Qubits).

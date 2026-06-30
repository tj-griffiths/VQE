# Ground State Preparation with VQE

**Thomas Griffiths** — Quantum Computing Course (PH10110), University of Edinburgh

This project examines the ground state manifold of a 1D Heisenberg-XXZ spin
chain using the Variational Quantum Eigensolver (VQE), simulated and on real
IBM Quantum hardware. The goal is to locate the model's two quantum phase
transitions from its ground states across the anisotropy range $J_z \in [-3, 3]$.

## The physics

$$H = \sum_{i=1}^{L} \left(X_i \otimes X_{i+1} + Y_i \otimes Y_{i+1} + J_z\, Z_i \otimes Z_{i+1}\right)$$

where $X_i, Y_i, Z_i$ are the Pauli matrices acting on site $i$, and $J_z$ is
the anisotropy term controlling the relative strength of the $ZZ$ interaction
against the $XX$ and $YY$ terms. This is a simplified model of a magnetic
insulator. The model has two quantum phase transitions as $J_z$ varies — one
at $J_z = 1$ and one at $J_z = -1$ — separating different magnetic
configurations. The transition at $J_z = -1$ is a
Berezinskii–Kosterlitz–Thouless (BKT) transition, which is continuous in
energy and requires a larger system size and more careful order-parameter
tracking to resolve than the transition at $J_z = 1$.

The antiferromagnetic structure factor $S(\pi)$ is used throughout as an
order parameter: it is near zero in the ferromagnetic/XY phases and
non-zero in the antiferromagnetic phase, making it sensitive to exactly
the kind of magnetic reordering this Hamiltonian produces.

## The notebooks

The four notebooks form a pipeline, each building on the previous one's
output, roughly in this order:

1. **`Introduction.ipynb`** — narrative walkthrough of the physics and method.
   Defines the Hamiltonian and $S(\pi)$ operator, computes an exact classical
   benchmark, and runs an initial VQE pass (EfficientSU2/SPSA) comparing
   noiseless vs. noisy simulation, then a fuller $J_z$ sweep with COBYLA and
   warm-started parameters, and a comparison of three ansatz families
   (EfficientSU2, RealAmplitudes, N-Local) against the exact benchmark. Start
   here to understand the approach.

2. **`Sweep.ipynb`** — the systematic version of the above. A script-style
   notebook (minimal narrative, mostly batch-job cells) that runs every
   combination of 3 ansatz × 2 repetition counts × 4 optimizers, first
   noiselessly at 8 qubits, then with a realistic noise model across 4, 6,
   and 8 qubits (72 configurations total). Exports a flattened CSV of
   per-configuration, per-$J_z$ results to the `Results/` directory.

3. **`Regression.ipynb`** — trains a Gaussian Process Regressor on the
   `Sweep.ipynb` output to extrapolate expected VQE performance (mean
   absolute energy error) to qubit counts the sweep never actually tested
   (10–20 qubits), with an explicit reliability horizon marking where that
   extrapolation should no longer be trusted. Used to choose the
   configuration for the real hardware run: **COBYLA with RealAmplitudes at
   12 qubits**.

4. **`IBM.ipynb`** — runs that chosen configuration on real IBM Quantum
   hardware (`ibm_fez`), one $J_z$ value at a time (hardware time is
   limited), using a physically-motivated warm start (Néel,
   ferromagnetic, or uniform-superposition initial state depending on the
   target phase). Once all `Jz` runs are collected, produces the final
   comparison: IBM hardware vs. noiseless simulation vs. noisy simulation
   vs. exact diagonalization.

## Headline results

The phase transition at $J_z = 1$ is visible even at small system sizes
(8 qubits); the BKT transition at $J_z = -1$ is much harder to resolve, as
expected for a continuous transition, and is more reliably located with
larger systems. Across the configuration sweep, **COBYLA generally
outperformed SPSA** once the ansatz repetition count was increased,
converging in far fewer iterations. The Gaussian Process Regression
extrapolation identified **RealAmplitudes with COBYLA at 12 qubits** as the
best-performing configuration in the size range relevant to the real
hardware budget, and this is the configuration used for the `IBM.ipynb` runs.

## Repository layout

```
Introduction.ipynb
Sweep.ipynb
Regression.ipynb
IBM.ipynb
Results/
    multiqubit_sweep_results2.csv   # Sweep.ipynb output, read by Regression.ipynb
    vqe_IBM_Jz{-3..3}_n12.npy        # one file per IBM hardware run
    comparison_energy_n12.png
    comparison_spi_n12.png
    vqe_IBM_convergence_Jz{...}_n12.png
```

`Results/` is a fixed save/load directory used consistently across
`Sweep.ipynb` and `IBM.ipynb` — both notebooks write their outputs there, and
`Regression.ipynb` reads the sweep CSV from there in turn. It already
contains the populated sweep results and IBM hardware run outputs (`.npy`
data plus saved plots) used in the final report, so `Sweep.ipynb` and
`IBM.ipynb` don't need to be rerun from scratch to reproduce
`Regression.ipynb` or the final comparison plots.

## Requirements

`qiskit`, `qiskit-aer`, `qiskit-algorithms`, `qiskit-ibm-runtime`, `numpy`,
`scipy`, `pandas`, `matplotlib`, `scikit-learn`

## Running

Each notebook's own header documents its specific requirements and data
dependencies (see each notebook for details). In short:

- `Introduction.ipynb` runs standalone, no external data or credentials
  needed. This notebook also doubles as the conceptual draft for the VQE
  background and method-comparison sections of the final written report.
- `Sweep.ipynb` needs a `token.json` (`{"api_key": "..."}`) with an IBM
  Quantum API key to authenticate `QiskitRuntimeService`, though it doesn't
  use real hardware time itself; it saves
  `Results/multiqubit_sweep_results2.csv`.
- `Regression.ipynb` reads that CSV from `Results/multiqubit_sweep_results2.csv`
  — already present in the repository from the sweep that produced the
  reported results.
- `IBM.ipynb` needs the same `token.json` and is run **once per $J_z$ value**
  manually (see the notebook's own workflow section) — it isn't a single
  start-to-finish run like the others. The `Results/` folder already
  contains the seven completed runs ($J_z \in \{-3,...,3\}$) used in the
  final comparison plots.

## Known limitations / future work

- `IBM.ipynb` has no automated loop over $J_z$ values; each hardware run
  requires manually editing and re-executing the first half of the notebook,
  which is intentional given the tight hardware time budget but means the
  notebook isn't a single linear "run all" the way the others are.
- The BKT transition at $J_z = -1$ is, by its nature as a continuous
  transition, only partially resolved at the system sizes used here (up to
  12 qubits on hardware); larger systems would sharpen it further but cost
  more circuit depth and hardware time.
- The Gaussian Process extrapolation in `Regression.ipynb` is only reliable
  out to roughly one kernel length-scale beyond the 8-qubit training
  boundary — predictions further out (toward 20 qubits) are explicitly
  flagged as low-confidence in that notebook's own plots rather than
  presented as equally trustworthy.

## Acknowledgements

Project specification: *PH10110: Quantum Computing Project — State
Preparation*, Dr Steven J. Thomson, University of Edinburgh, February 2026.
Some sections of `Introduction.ipynb` were collaboratively developed with a
project teammate (noted inline where applicable).
# Noise Robustness of a Quantum Circuit for Two-Flavor Neutrino Oscillation

Independent research project by Muhammad Abdullah Anwar, August 2026.
ORCID: [0009-0009-2691-5229](https://orcid.org/0009-0009-2691-5229)

## What this is

This project investigates whether a minimal single-qubit quantum circuit can accurately reproduce two-flavor neutrino oscillation probabilities, and quantifies how much noise the circuit can tolerate before it stops reliably showing the oscillation pattern — both in simulation and on real IBM Quantum hardware.

**Research question:** At what combined noise level does a single-qubit quantum circuit stop reliably recovering the two-flavor neutrino oscillation pattern, and by what margin does this exceed real hardware noise levels?

**Key results:**
- The circuit exactly reproduces the analytical oscillation formula (MAE ≈ 6.8×10⁻¹⁷, floating-point precision) using an exact statevector simulation.
- Shot (sampling) noise follows the theoretical 1/√n scaling law, validated across 100–10,000 shots and 5 random seeds.
- The circuit's oscillation pattern remains reliably recoverable (≥50% contrast retention) up to a combined noise severity of 1.143 ± 0.017 (5-seed average) — a robustness margin of roughly **28×–50× (centered ~38×)** over current real hardware noise levels, depending on the failure-cutoff definition used.
- Real hardware error mitigation (dynamical decoupling, gate twirling) was tested across 4 repeated trials on a single pinned backend (ibm_marrakesh); a statistical test found **no significant effect** (p = 0.362), reversing an initial single-trial result that had suggested a modest improvement — illustrating why single trials are unreliable evidence on noisy NISQ hardware.

Full details, methodology, and discussion are in the accompanying paper.

## Repository contents

| File | Description |
|---|---|
| `01_Classical_Neutrino_Oscillation.ipynb` | Analytical two-flavor oscillation formula and baseline plot |
| `02_Ideal_Qubit_Simulation.ipynb` | Exact statevector circuit verification, shot-noise scaling study (5-seed averaged), circuit diagram, L/E equivalence test |
| `03_Noise_Shots_Hardware.ipynb` | Noise-type comparison (stress-test and realistic-calibration levels), combined-noise failure threshold with cutoff sensitivity, real IBM hardware execution, repeated-trial error mitigation analysis |
| `Research_Log_Neutrino_Quantum_Project.md` | Full chronological research log — every finding, correction, and methodological lesson learned along the way |
| `Quantum_Neutrino_Oscillation_Research_Paper.docx` | Full write-up (paper format), including all figures and references |

## How to run

1. Open any notebook in [Google Colab](https://colab.research.google.com/)
2. Run cells from top to bottom, in order — each notebook installs its own dependencies (`qiskit`, `qiskit-aer`, `qiskit-ibm-runtime`) in its first cell
3. Notebook 03's real-hardware cells require a free [IBM Quantum Platform](https://quantum.ibm.com/) account and API token. **Do not re-run the job-submission cells** — they are commented out / disabled in the final notebook. To retrieve the original results without submitting new hardware jobs, use `service.job(existing_job_id)` with the job IDs recorded in the notebook and research log.

## Physics parameters used

- Channel: atmospheric (νμ ↔ ντ)
- θ₂₃ = 43.3° (0.7557 rad), Δm²₃₁ = 2.5×10⁻³ eV²
- Source: NuFit 6.0 (2024), Esteban et al., *JHEP* 12 (2024) 216 — specifically the fit including Super-Kamiokande atmospheric data, first octant, normal mass ordering. See [nu-fit.org](http://www.nu-fit.org/).

## Real hardware backends used

- `ibm_fez` — initial single-trial validation
- `ibm_marrakesh` — repeated 4-trial statistical mitigation analysis (backend switched due to queue availability; all trials within each individual comparison ran on a single, fixed backend)

## Requirements

Python 3.12.13, Qiskit 2.5.1, qiskit-aer, qiskit-ibm-runtime, numpy, matplotlib, scipy. All installed automatically within each notebook's first cell.

## Citation

If referencing this work, please cite the accompanying paper (see `Quantum_Neutrino_Oscillation_Research_Paper.docx`). A DOI via Zenodo will be added here once archived.

## License

This project is shared for educational and research-transparency purposes. Feel free to reuse or build on it with attribution.

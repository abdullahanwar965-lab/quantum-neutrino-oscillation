# Research Log: Quantum Circuit Simulation of Two-Flavor Neutrino Oscillation

This document records the physics reasoning, methodological development, and validation steps behind the project, in the order they were established. It serves as supplementary material to the accompanying paper, providing a transparent account of how each result was reached, including corrections made along the way.

---

## 1. Physical Background

**Neutrino oscillation.** Neutrinos are produced and detected in one of three flavor states (electron, muon, tau) but propagate as a superposition of mass states, which accumulate different quantum phases over the course of travel. This causes the flavor composition to oscillate periodically with the ratio of distance traveled to energy, L/E. The phenomenon was historically motivated by the solar neutrino problem: early detectors observed roughly one-third of the predicted electron-neutrino flux from the Sun. This deficit is not an unresolved anomaly — the "missing" neutrinos had changed flavor en route. Resolving it, however, required more than the vacuum oscillation formula used in this project: the solar L/E scale (Earth-Sun baseline ~10⁸ km, MeV-scale energies) is many orders of magnitude larger than the atmospheric case studied here (L/E up to ~2000 km/GeV) — large enough that the vacuum oscillation term averages to a near-constant value rather than tracing a clean sinusoid — and for the higher-energy solar neutrinos most precisely measured (e.g., by SNO), flavor conversion is in fact dominated by the Mikheyev–Smirnov–Wolfenstein (MSW) matter effect, a resonance driven by the Sun's density gradient, rather than by vacuum oscillation alone. The clean oscillatory pattern this project's circuit reproduces belongs to the atmospheric channel, a different L/E regime in which vacuum oscillation is the appropriate and dominant description.

**Two-flavor oscillation formula:**

P = sin²(2θ) · sin²(1.27 · Δm² · L/E)

- sin²(2θ) sets the amplitude (peak height) of the oscillation, governed by the mixing angle and independent of distance.
- sin²(1.27 · Δm² · L/E) is the oscillating term, where L is distance traveled, E is neutrino energy, and Δm² is the mass-squared splitting between mass states — the physical origin of oscillation itself.

**Qubit representation.** A qubit's superposition, describable by a single mixing angle, provides a natural analogue for the two-flavor mixing structure. The circuit implements this as three gates: RY(2θ) prepares the mixed state (flavor mixing), RZ(φ), with φ derived from L, E, and Δm², evolves the phase (propagation), and RY(−2θ) inverts the initial rotation before measurement, yielding the oscillation probability directly from measurement statistics.

---

## 2. Circuit Validation Against Theory

The three-gate circuit (RY(2θ) → RZ(φ) → RY(−2θ) → measure) was first validated against the classical formula using Qiskit's exact statevector simulator (`Statevector.from_instruction`), which computes the quantum state with no sampling randomness.

**Result:** MAE = 6.80 × 10⁻¹⁷, maximum error = 3.33 × 10⁻¹⁶ — both at floating-point precision. This confirms the circuit design reproduces the analytical formula exactly, independent of any statistical sampling effects, and establishes the exact-simulator result as the baseline against which all subsequent noise and hardware comparisons are measured.

**Parameter choice.** Atmospheric-channel parameters were used throughout: θ₂₃ = 43.3° (0.7557 rad), Δm²₃₁ = 2.5 × 10⁻³ eV², normal mass ordering, from NuFit 6.0 (Esteban et al., 2024). This channel is physically consistent with the L range used (0–2000 km), appropriate for atmospheric neutrinos.

---

## 3. Statistical Validation of Sampling Noise

With the exact baseline established, shot-based sampling was characterized to confirm that residual error in the finite-shot simulation is governed entirely by standard statistical sampling noise, not by any circuit-design flaw.

Shot counts of 100, 1000, 8000, and 10000 were each evaluated against the exact statevector result, averaged over five fixed random seeds (1–5) for stability:

| Shots | MAE |
|---|---|
| 100 | 0.0257 ± 0.0082 |
| 1000 | 0.0076 ± 0.0037 |
| 8000 | 0.0026 ± 0.0009 |
| 10000 | 0.0023 ± 0.0005 |

Scaling ratios closely tracked the theoretical 1/√n prediction (e.g. 100→1000 shots: observed ratio 3.37 vs. theoretical √10 ≈ 3.16), with agreement tightening as shot count increased. Combined with the floating-point-precision exact-circuit match, this confirms the sampled simulation's residual error is governed by standard sampling statistics alone.

**L/E dependence.** Four (L, E) pairs sharing the ratio L/E = 500 km/GeV — (500, 1.0), (1000, 2.0), (250, 0.5), (2500, 5.0) — were confirmed to produce the identical exact probability (0.99620475) to eight decimal places, verifying that the oscillation probability depends only on the ratio L/E, as required by the physics.

---

## 4. Noise-Type Characterization

Three standard noise mechanisms — depolarizing, amplitude damping, and phase damping — were compared at three severity levels: two exaggerated "stress test" levels (5%, 15% per-gate error) and one "realistic calibration" level (~0.3% per-gate error, 1% readout error), reflecting typical published superconducting-qubit specifications.

**Results (MAE vs. exact statevector baseline):**

| Configuration | Depolarizing | Amplitude damping | Phase damping |
|---|---|---|---|
| Stress test (5%) | 0.0403 | 0.0318 | 0.0122 |
| Stress test (15%) | 0.1182 | 0.0984 | 0.0426 |
| Realistic calibration | 0.0067 | 0.0054 | 0.0051 |

The damage ranking (depolarizing worst, amplitude damping intermediate, phase damping mildest) held consistently across all three noise levels. Amplitude damping's asymmetric signature — larger peak suppression than trough elevation — persisted even at realistic noise levels (e.g. peak suppression 0.0147 vs. trough elevation 0.0110), consistent with its physical origin as one-directional energy relaxation toward the ground state. At realistic hardware-calibration noise levels, MAE remained below 0.007 — an order of magnitude smaller than under stress-test conditions — indicating that current-generation quantum hardware introduces only modest distortion to this signal.

---

## 5. Noise-Robustness Threshold Analysis

To move beyond a qualitative noise comparison, the project defined a quantitative research question: at what combined noise level does the circuit stop reliably reproducing the oscillation pattern, and by what margin does this threshold exceed real hardware noise levels?

**Failure metric.** Contrast retention — (measured peak − measured trough) / (ideal peak − ideal trough) — was adopted over MAE (which conflates noise types) or R² (which can remain high even after amplitude collapse), since it directly measures oscillation visibility. Failure was defined as contrast retention falling below 50%.

**Combined noise model.** Depolarizing (10% × severity), amplitude damping (10% × severity), and phase damping (5% × severity) were applied simultaneously per gate, plus scaled readout error, representing realistic hardware where all noise mechanisms act together.

The severity sweep was repeated across five independent random seeds to quantify threshold uncertainty. The 50% contrast-retention threshold, computed separately per seed, was: 1.174, 1.130, 1.135, 1.130, 1.145 — **mean 1.143 ± 0.017**, a tight spread confirming the threshold is a stable, reproducible feature of the circuit and noise model rather than a single-seed artifact.

Sensitivity to the failure-cutoff definition was tested directly: a 40% cutoff yields a threshold severity of 1.487; 50% yields 1.142; 60% yields 0.855.

**Real-hardware margin — calibration-dependent, not a single number.** The severity scale is a single dial that fixes the relative proportions of gate error and readout error together; real hardware does not necessarily preserve those proportions. Calibrating severity against the ~0.3% per-gate error assumed for "realistic calibration" gives severity ≈ 0.03, and a margin of roughly 28–50× (centered ≈38×). Calibrating severity instead against the directly measured readout error from Section 7 (1.40% on ibm_marrakesh) gives severity ≈ 0.90 — close to the failure threshold itself, implying a margin of only ≈1.3×. These two calibrations disagree roughly 30-fold because the single-parameter model cannot represent "low gate error, high readout error" and "high gate error, low readout error" simultaneously. The threshold itself, in the model's own units, is a robustly-estimated quantity (stable across 5 seeds); converting it into a specific real-hardware safety-margin multiplier is not, and any single figure should be read as approximate.

---

## 6. Real-Hardware Validation

**Single-run illustration.** The corrected circuit was executed on real IBM Quantum hardware (backend: `ibm_fez`, IBM Quantum Platform Open Plan) across 8 L/E points at 1000 shots each, yielding MAE = 0.0092 against the exact statevector prediction — closely tracking the realistic-noise-model prediction from Section 4 (MAE 0.005–0.007), providing external validation of the simulated noise analysis.

This establishes the project's three-tier validation structure: (1) the analytical formula, (2) the ideal simulation matching it to floating-point precision, and (3) real hardware confirming the full pipeline end-to-end.

---

## 7. Real-Hardware Error Mitigation Study

**Methodological note.** An initial attempt at comparing raw versus error-mitigated hardware execution used automatic backend selection (`service.least_busy()`), which silently assigned different physical machines to separate submissions, confounding the comparison with an uncontrolled hardware change. This was identified and corrected by pinning the backend explicitly for all runs in a given comparison. The job-submission cell was also disabled immediately after use, since re-running a notebook top-to-bottom would otherwise resubmit real hardware jobs unnecessarily. Both issues are noted here as they informed the experimental controls used in all subsequent hardware comparisons and are documented in the accompanying code repository.

**Single-trial result** (backend `ibm_fez`, 20 L/E points, 4000 shots, readout error 0.0140): raw MAE 0.0099, mitigated MAE (dynamical decoupling + gate twirling) 0.0095 — a 3.7% apparent improvement.

**Repeated-trial result.** A single trial is insufficient evidence on noisy NISQ hardware, so the comparison was repeated across 4 independent trials (backend `ibm_marrakesh`, pinned throughout; 10 L/E points, 2000 shots per trial):

| Trial | Raw MAE | Mitigated MAE |
|---|---|---|
| 1 | 0.0121 | 0.0156 |
| 2 | 0.0122 | 0.0153 |
| 3 | 0.0152 | 0.0113 |
| 4 | 0.0119 | 0.0142 |
| **Mean ± SD** | **0.0129 ± 0.0014** | **0.0141 ± 0.0017** |

Each mitigated trial used the same transpiled circuits and L/E points as its corresponding raw trial, submitted as a matched pair on the same pinned backend — a paired design. Paired t-test: t = −0.721, p = 0.523 — **not statistically significant**.

The mean mitigated MAE is slightly *higher* than raw, the opposite direction from the single-trial result, but the difference is well within the trial-to-trial spread. **Conclusion: dynamical decoupling and gate twirling show no statistically significant effect, positive or negative, on this circuit's real-hardware accuracy.** The original single-trial improvement was a favorable draw from a noisy system rather than a reliable effect, underscoring that single-trial comparisons on shared cloud quantum hardware are not sufficient to establish a mitigation technique's effectiveness for a shallow circuit of this kind.

This null result is close to the physically expected outcome for gate twirling specifically: Qiskit's Sampler-level gate twirling is designed to convert coherent errors into stochastic ones by randomizing two-qubit gates, and this circuit contains no two-qubit gates at all, so gate twirling has essentially no error to act on here. Dynamical decoupling, which fills idle time during two-qubit-gate execution elsewhere on the chip, is similarly of limited relevance to a single active qubit running a 3-gate circuit with little idle time. Both were included as the default, minimal-cost Sampler-level options, not because they were expected to meaningfully help a circuit of this shape — the null result is consistent with that expectation, not a surprising failure of the techniques.

Raw per-trial, per-L/E-point measurement data (not just job IDs) for this analysis is archived alongside the notebook as `hardware_repeated_trials_raw_data.csv`, generated by a dedicated export cell added to the notebook.

---

## 8. Related Work and Literature Positioning

**Argüelles, C. A., & Jones, B. J. P. (2019).** *Neutrino oscillations in a quantum processor.* Physical Review Research, 1(3), 033176. arXiv:1904.10559.
Demonstrated both two-flavor (single-qubit) and three-flavor (two-qubit) quantum circuit simulations of neutrino oscillation, establishing the foundational approach this project adopts for the two-flavor case.

**Joshi, S., Rajpoot, G., & Shukla, P. (2025).** *Quantum circuits for simulating neutrino propagation in matter.* Physica Scripta, 100, 085111. arXiv:2503.20238.
Closely related methodology: single-qubit two-flavor circuits for neutrino propagation through matter, validated on both simulators and real IBM hardware. The most directly comparable prior work to this project's core simulator-plus-hardware validation approach, though focused on matter-propagation effects rather than the noise-robustness question investigated here.

**Joshi, S. (2026).** *Quantum simulation of neutrino oscillations with bosonic encoding.* arXiv:2606.18755.
The same lead author's subsequent work exploring an alternative bosonic/cavity-mode (Fock-basis) encoding via SNAP and displacement gates, illustrating the range of active encoding approaches in this field.

**Nguyen, H. C., Bach, B. G., Nguyen, T. D., Tran, D. M., Nguyen, D. V., & Nguyen, H. Q. (2023).** *Simulating neutrino oscillations on a superconducting qutrit.* Physical Review D, 108, 023013. arXiv:2212.14170.
An alternative encoding using a single three-level qutrit rather than multiple qubits to represent three flavors directly, validated on real hardware.

**Turro, F., Chernyshev, I. A., Bhaskar, R., & Illa, M. (2025).** *Qutrit and qubit circuits for three-flavor collective neutrino oscillations.* Physical Review D, 111, 043038. arXiv:2407.13914.
Explores qutrit-based encodings for the three-flavor collective-oscillation case.

**Singh, G., Arvind, & Dorai, K. (2025).** *Simulating three-flavor neutrino oscillations on a nuclear magnetic resonance quantum processor.* Physica Scripta, 100, 085106. arXiv:2412.15617.
Three-flavor simulation on nuclear magnetic resonance hardware, illustrating an alternative physical platform to superconducting qubits.

**Spagnoli, L., et al. (2025).** *Collective neutrino oscillations in three flavors on qubit and qutrit processors.* Physical Review D, 111, 103054. arXiv:2503.00607.
Presents qubit and qutrit encodings for the full three-flavor collective-oscillation system.

**Hall, B., Roggero, A., Baroni, A., & Carlson, J. (2021).** *Simulation of collective neutrino oscillations on a quantum computer.* Physical Review D, 104, 063009. arXiv:2102.12556.

**Bleau, K., Ilic, N., Kopp, J., Rahaman, U., & Yu, X. Y. (2026).** *Quantum Simulation of Collective Neutrino Oscillations using Dicke States.* arXiv:2604.07452.
Introduces a qubit-efficient Dicke-state-based encoding for dense neutrino gas simulations.

**Esteban, I., Gonzalez-Garcia, M. C., Maltoni, M., Martinez-Soler, I., Pinheiro, J. P., & Schwetz, T. (2024).** *NuFit-6.0: updated global analysis of three-flavor neutrino oscillations.* Journal of High Energy Physics, 12, 216. nu-fit.org.
Source of the θ₂₃ and Δm²₃₁ parameters used throughout this project.

**Rhyno, B., Majumder, S., Vishveshwara, S., & Najafi, K. (2025).** *Quantum critical dynamics and emergent universality in decoherent digital quantum processors.* arXiv:2512.13143.
Not neutrino-specific, but methodologically relevant: uses the same Sampler-native error-suppression options (gate twirling, dynamical decoupling) as this project, chosen as a minimal-cost mitigation approach for studying inherent hardware noise — consistent with the technique used in Section 7 above.

### Positioning

This body of work establishes that quantum circuits can accurately reproduce neutrino oscillation probabilities, generally validated against theory and, in several cases, real hardware. This project adopts the same minimal single-qubit two-flavor approach for its exact verifiability, but extends beyond reproduction to address a quantitative robustness question not emphasized in this literature: at what combined noise level does the circuit stop reliably recovering the oscillation pattern, and by what safety margin does this threshold exceed real current-hardware noise levels? This project further reports a directly verified, controlled comparison of raw versus error-mitigated real-hardware execution, including a transparent account of a methodological pitfall encountered and corrected during real-hardware experimentation — a degree of process detail intended to support reproducibility, and reported here even though it did not favor a clean, more attractive-looking result.

---

## 9. Methodological Notes

The following corrections, made during the course of the project, are recorded here as part of the scientific record:

- An early analysis mixed oscillation parameters from two different physical channels (the mixing angle from the solar channel with the mass-squared splitting from the atmospheric channel). This was identified and corrected by switching consistently to the atmospheric channel (θ₂₃, Δm²₃₁) throughout, matching the L range used in the study.
- An initial "ideal" baseline used a high shot count (8000) rather than an exact statevector calculation, meaning it still carried residual sampling noise rather than representing a true noiseless reference. This was corrected by adopting Qiskit's exact statevector simulator as the baseline (Section 2).
- Automatic backend selection during hardware experimentation was found to silently assign different physical machines across separate job submissions, confounding an early raw-versus-mitigated comparison. This was corrected by pinning the backend explicitly for all runs within a given comparison (Section 7).
- The initial single-trial hardware mitigation result (Section 7) was re-tested across four independent trials and found not to be statistically significant, reversing the original single-trial conclusion. This is reported as a substantive finding in its own right, illustrating that mitigation-technique effectiveness cannot be established from a single trial on noisy, shared quantum hardware.

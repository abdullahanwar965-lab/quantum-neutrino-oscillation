# Research Log — Quantum Simulation of Neutrino Oscillation

Running notes of physics concepts understood and findings confirmed, kept as the project progresses. This becomes the raw material for the paper's Background, Method, Results, and Discussion sections later.

---

## Core Physics Understood

**Neutrino oscillation basics**
- Neutrinos come in three flavors (electron, muon, tau).
- A neutrino created in one flavor is a mixture of underlying mass states, which evolve with different phases as it travels. This causes it to be detected as a different flavor later — oscillation.
- Historical motivation: the solar neutrino problem — early detectors saw ~1/3 the expected electron-neutrino flux from the Sun. Not a violation of physics; the "missing" neutrinos had oscillated into flavors those detectors couldn't see. Oscillation is the accepted, confirmed explanation, not an open discrepancy.

**Two-flavor oscillation formula**
P = sin²(2θ) · sin²(1.27·Δm²·L/E)
- sin²(2θ): sets the *ceiling* (max amplitude) of the oscillation — how strongly the two flavors mix. Independent of distance.
- sin²(1.27·Δm²·L/E): the actual oscillating term. L = distance traveled, E = neutrino energy, Δm² = mass-squared difference between mass states (the physical reason oscillation exists at all — if masses were equal, no oscillation).
- Confirmed experimentally (via code): lower E → shorter wavelength (more oscillations per distance); higher E → longer wavelength. Peak height controlled only by θ, unaffected by E or L.
- θ vs peak height: increasing, but not linear forever — sin²(2θ) actually peaks at θ=45° then decreases. Real neutrino mixing angles stay well below that, so this ceiling behaves as "the bigger θ, the bigger the peak" in the practically relevant range.

**Qubit representation of oscillation**
- A qubit isn't binary — it holds a "blend" of 0 and 1, describable by an angle. Measurement collapses it to a definite outcome, with probability depending on that angle.
- Circuit logic: RY(2θ) prepares the mixed state (mirrors "flavor mixing"), RZ(phase from L/E/Δm²) evolves it (mirrors "traveling"), RY(−2θ) undoes the prep, then measurement gives the oscillation probability.
- Confirmed: ideal (noiseless) qubit circuit output closely matches the classical formula across the full L range.

---

## Findings Log (chronological)

**Finding 1 — Ideal circuit matches theory**
The 1-qubit circuit (RY → RZ → RY⁻¹ → measure) reproduces the classical two-flavor oscillation curve closely across L = 0–2000 km, confirming the circuit design is physically correct.

**Finding 2 — Shot noise, not circuit error, causes residual mismatch**
Small gaps between the ideal quantum circuit's output and the classical formula are due to statistical sampling noise (finite shots), not a flaw in the circuit. Confirmed by increasing shots from 2000 → 8000: max error dropped from ~0.022 to ~0.012, roughly matching the expected 1/√(shots) scaling.

**Finding 3 — Shot noise is theoretically well-predicted**
Measured error vs. classical probability value was compared against the theoretical shot-noise bound √(p(1−p)/shots). The measured scatter followed the predicted hill-shaped curve (largest near p=0.5, smallest near p=0 or p≈peak), confirming the error behaves as standard sampling statistics predict.

**Finding 4 — Shot-count comparison confirms 1/√shots scaling precisely**
Tested 100 / 1000 / 10000 shots directly:
- 100 shots: MAE = 0.0272
- 1000 shots: MAE = 0.0094 (ratio vs 100: 2.89×, theory predicts √10 ≈ 3.16×)
- 10000 shots: MAE = 0.0028 (ratio vs 1000: 3.36×, theory predicts √10 ≈ 3.16×)
Both ratios closely track the theoretical √10 prediction — strong quantitative confirmation.

**Finding 5 — Noise type comparison (depolarizing, amplitude damping, phase damping) at 15% strength**
- Depolarizing: highest overall MAE. Symmetric pull toward 0.5 (peak −0.133, trough +0.191) — matches its definition as "mix in a random 50/50 outcome with probability p."
- Amplitude damping: most damaging to peaks specifically (−0.218) but barely touches troughs (+0.026). Matches its physical meaning (energy relaxation toward |0⟩ only — asymmetric, one-directional).
- Phase damping: mildest overall (peak −0.060, trough +0.063). Corrupts phase/timing information before it's converted to probability, one step removed from the final readout — explains why it's less directly damaging than the other two.

**Important framing note (recorded to avoid a real mistake):**
This noise study is NOT a claim that real neutrinos experience decoherence in space. It models imperfections in the quantum *computer* being used as a calculator for the physics formula. The research question is about the reliability of NISQ-era quantum hardware as a computational tool for particle physics — not about physical neutrino decoherence.

---

## Corrections / Self-Corrected Mistakes (worth keeping — shows real scientific process)
- Initially mis-stated that shot-noise error should NOT peak near p=0.5 after seeing a noisy raw scatter plot; re-checked against the theoretical formula and confirmed the original prediction (error peaks near p=0.5) was correct. Individual noisy runs can look misleading — theoretical/statistical validation caught this.
- Initially misread which visual result belonged to which shot count (E=0.5 vs E=2.0) — corrected by re-examining the graphs directly rather than relying on memory.

---

## Checkpoint 1 Review — External Feedback and Corrections

A detailed external review of the original single-notebook project identified several real issues that needed fixing before continuing. Key corrections made:

**Issue: "Ideal" result wasn't actually ideal**
Original "ideal" comparison used 8000 *shots*, which still carries sampling noise — not a true noiseless baseline. Fixed by using an exact statevector simulator (Qiskit `Statevector.from_instruction`), which computes the exact quantum state with no randomness at all.

**Issue: Mismatched oscillation channel parameters**
Original params (θ=0.6 rad, Δm²=2.5×10⁻³ eV²) accidentally combined the mixing angle from the *solar* channel (θ₁₂) with the mass-squared splitting from the *atmospheric* channel (Δm²₃₁) — two different physical oscillation channels stitched together inconsistently.
Fixed: switched to the atmospheric channel (ν_μ ↔ ν_τ) consistently:
- θ₂₃ = 43.3° = 0.7557 rad
- Δm²₃₁ = 2.5×10⁻³ eV²
- Mass ordering: normal ordering
- Source: NuFIT 6.0 (2024), Esteban et al., JHEP 12 (2024) 216, nu-fit.org
This channel choice also matches the L range used (0–2000 km), which is physically appropriate for atmospheric neutrinos (solar neutrinos travel ~150 million km, so the original L range wouldn't make sense for that channel).

**Issue: Plotted against L instead of L/E**
Since E was held fixed at 1 GeV throughout, L and L/E were numerically identical, masking the fact that L/E is the physically correct independent variable. Fixed: x-axis is now L/E (km/GeV) explicitly, with a note in the notebook that the formula's constant 1.27 already assumes L in km, E in GeV, Δm² in eV² (no unit conversion needed).

**Finding 6 — Exact statevector vs analytical formula (corrected Phase 1)**
Using Qiskit 2.5.1, `Statevector.from_instruction` on the 3-gate circuit (RY(2θ) → RZ(φ) → RY(−2θ)):
- MAE = 6.80 × 10⁻¹⁷
- Max error = 3.33 × 10⁻¹⁶
Both at floating-point precision, confirming the circuit design is exactly correct — not merely "close" due to sampling luck. This properly isolates circuit correctness from sampling noise, addressing the reviewer's central concern.

**Other issues flagged for future fixes (not yet addressed):**
- Noise-type comparison (depolarizing/amplitude/phase damping) used equal *numerical* parameters (0.15 each), which does not represent equal *physical* noise strength across channel types — conclusion about "which noise type is worst" needs either an equal-infidelity comparison or an explicit "illustrative only" caveat.
- "Theoretical shot noise bound" should be renamed "expected one-sigma shot uncertainty" — it's a standard deviation estimate, not a strict bound.
- Noise strengths tested (5%, 15%) are much higher than real IBM gate error rates (~0.1–1.5%) — need labeling as stress tests, plus a separate run closer to real calibration data.
- Bug: noise comparison graph and printed peak/trough numbers were generated from two separate (re-randomized) simulation runs — need to store one array and reuse it for both.
- No fixed random seed yet — results not currently reproducible run-to-run.
- Project should be split into 3 notebooks per plan (01 Classical, 02 Ideal Qubit, 03 Noise/Shots/Hardware) instead of one combined notebook.
- Need scientific interpretation paragraphs after each figure, not just graphs/numbers.

**Finding 7 — Shot-noise scaling validated against exact baseline, averaged over 5 seeds**
Redid the shot-count comparison (100/1000/8000/10000) against the *exact* statevector result (not the analytical formula directly), with a fixed seed list [1,2,3,4,5] averaged for stability:
- 100 shots: MAE = 0.0257 ± 0.0082
- 1000 shots: MAE = 0.0076 ± 0.0037
- 8000 shots: MAE = 0.0026 ± 0.0009
- 10000 shots: MAE = 0.0023 ± 0.0005

Scaling ratios closely matched theoretical √n predictions:
- 100→1000: actual 3.37 vs theoretical 3.16
- 1000→8000: actual 2.90 vs theoretical 2.83
- 8000→10000: actual 1.15 vs theoretical 1.12

Agreement with theory tightens as shot count increases, consistent with larger samples themselves being less noisy estimates. Combined with Finding 6 (exact circuit match to floating-point precision), this confirms residual error in the sampled simulation is governed entirely by standard sampling statistics, not by any flaw in circuit design.

**Key summary sentence for paper:** "Averaged over 5 random seeds, measurement error scaled with shot count in close agreement with the theoretical 1/√n prediction, confirming that residual error in the sampled simulation is governed by standard statistical sampling noise rather than any flaw in the circuit design, which was independently verified to match the analytical formula to floating-point precision (MAE ~10⁻¹⁷)."

**Finding 8 — L/E equivalence confirmed**
Tested four (L, E) pairs sharing the same ratio L/E = 500 km/GeV: (500,1.0), (1000,2.0), (250,0.5), (2500,5.0). All four produced the identical exact probability (0.99620475) to 8 decimal places, confirming the oscillation probability depends only on the ratio L/E, not on L or E individually — as required by the physics, and as now correctly reflected by plotting against L/E rather than L alone.

**Finding 9 — Noise comparison rebuilt correctly (Notebook 03)**
Rebuilt the noise-type comparison fixing all issues flagged in the checkpoint review: results stored once and reused for both plots and printed numbers (no regeneration bug), fixed seed throughout, readout noise included, and three clearly labeled configurations tested — two exaggerated "stress tests" (5%, 15% gate error) plus one "realistic calibration" run (~0.3% gate error + 1% readout error, closer to real current IBM hardware specs).

Results (MAE vs exact statevector baseline):

| Config | Depolarizing | Amplitude damping | Phase damping |
|---|---|---|---|
| Stress 5% | 0.0403 | 0.0318 | 0.0122 |
| Stress 15% | 0.1182 | 0.0984 | 0.0426 |
| Realistic (~0.3%+1% readout) | 0.0067 | 0.0054 | 0.0051 |

Ranking (depolarizing worst, then amplitude damping, then phase damping) held consistently across all three noise levels, strengthening the qualitative conclusion beyond a single arbitrary parameter choice.

Amplitude damping's asymmetric signature (peak suppression >> trough elevation) persisted even at realistic noise levels, e.g. realistic config: peak suppression 0.0147 vs trough elevation 0.0110 — consistent with its physical origin as one-directional energy relaxation toward the ground state.

Key finding: at realistic hardware-calibration noise levels, MAE stayed below 0.007 — an order of magnitude smaller than under stress-test conditions (0.10–0.12 at 15%), suggesting current-generation quantum hardware would introduce only modest distortion to this signal.

Terminology corrected per review: "peak suppression" / "trough elevation" used instead of "shift," since these describe magnitude change at fixed positions, not displacement of the peak/trough location along the L/E axis.

**Finding 10 — Real IBM Quantum hardware validation**
Ran the corrected circuit (atmospheric channel parameters) on real IBM hardware (backend: ibm_fez) via the IBM Quantum Platform Open Plan, using 8 L/E test points and 1000 shots each.

Result: MAE = 0.0092 vs. exact statevector prediction. Visually, all 8 real-hardware points closely track the exact curve's shape, peak, and trough.

Notably, this real-hardware MAE (0.0092) is close to the realistic-noise-model prediction from Finding 9 (MAE 0.005–0.007 across depolarizing/amplitude/phase damping at calibration-level strengths) — meaning the noise simulation study was a reasonably good predictor of actual hardware behavior, providing an external validation of the simulated noise analysis, not just an isolated additional data point.

This completes the three-tier validation structure of the project:
1. Analytical formula (theory)
2. Ideal simulation — exact statevector match to floating-point precision (Finding 6)
3. Real hardware — confirms the full pipeline (theory → circuit → noise prediction → real device) end-to-end

This is likely the single strongest summary figure for the paper, demonstrating the complete research arc in one plot.

**Reframed Research Question (Checkpoint 2 review, point 2)**
Original project reproduced a known result (qubit matching classical formula) without a novel question. Reframed to: "At what combined noise level does the single-qubit circuit stop reliably recovering the oscillation pattern, and by what margin does this exceed real hardware noise levels?"

Defined failure via **contrast retention**: (measured peak − measured trough) / (ideal peak − ideal trough). Chosen over MAE (conflates noise types) or R² (can stay high even after amplitude collapses) because it directly measures oscillation visibility — a standard physics concept — and distinguishes "noisy but still oscillating" from "signal destroyed."
Failure threshold defined as contrast retention < 50%.

**Finding 11 — Combined noise model and failure threshold**
Built a combined noise model applying depolarizing (10%×severity), amplitude damping (10%×severity), and phase damping (5%×severity) simultaneously per gate, plus scaled readout error — representing realistic hardware where all noise mechanisms act together, not in isolation (addresses review point 6).

Swept severity from 0 to 1.5 (15 points):
- severity=0.00: retention=0.990
- severity=0.50 (≈mid stress-test level): retention≈0.73
- severity=1.00 (≈full single-type stress-test level): retention=0.558 (did not cross threshold)
- Extended sweep to severity=1.5 confirmed threshold crossing empirically at **severity ≈ 1.18** (linear extrapolation from the first sweep predicted ≈1.10 — close agreement, a useful consistency check)

**Key safety-margin result:** Realistic hardware noise (Finding 9, ~0.3% gate error) corresponds to severity ≈ 0.03 on this scale. Failure occurs at severity ≈ 1.18 — a margin of roughly **40× below the failure point**. This indicates the circuit design is robust with substantial margin relative to current real quantum hardware performance, not just "somewhat noise-tolerant."

This result reframes the project's core claim from "a qubit can reproduce this formula" (already known) to a specific, quantitative, testable finding about the noise-robustness of this particular circuit design.

**Finding 12 — Expanded hardware run with error mitigation (review points 5, 7) — verified result**

Initial attempts at this comparison used `service.least_busy()` to select a backend, which silently picked different physical machines across separate submissions (observed: ibm_marrakesh and ibm_fez on different runs), confounding the raw-vs-mitigated comparison with an uncontrolled backend change. One early trial also showed an unexplained large negative result (-24.8%) that could not be confidently attributed to a specific, verified job pair upon review.

**Corrected methodology:** backend was pinned explicitly (`service.backend("ibm_fez")`) rather than auto-selected, ensuring both raw and mitigated runs executed on identical hardware. The submission cell was run exactly once and immediately disabled to prevent accidental resubmission (a real risk encountered during this project — Colab's "Run all" will resubmit real hardware jobs if the submission cell is left active).

**Verified result** (ibm_fez, readout error 0.0140, 20 L/E points, 4000 shots):
- Raw MAE: 0.0099
- Mitigated MAE (dynamical decoupling + gate twirling): 0.0095
- Change: **+3.7% improvement**
- Sanity check: raw and mitigated probability arrays differ meaningfully (max diff 0.0182), confirming both are genuine independent hardware executions, not a duplicated or stale result

**Interpretation:** the small positive effect is consistent with the physical expectation that dynamical decoupling and gate twirling have limited impact on a shallow 3-gate, single-qubit circuit with minimal idle time and low accumulated coherent gate error — meaningful but modest improvement, not dramatic, and readout error (the likely dominant error source here) remains unaddressed by these particular techniques.

**Process note for the paper's methods/limitations section:** this result is reported alongside the methodological lesson learned — that backend auto-selection and unmanaged job-resubmission risk can silently confound hardware comparisons, and that explicit backend pinning plus sanity-checking (verifying result arrays are non-identical) are necessary controls for credible real-hardware experiments on shared cloud quantum platforms.

## Literature Review and Positioning (review point 8)

A search of current literature confirms quantum simulation of neutrino oscillation is an active, real research area with several established approaches. This project's methodology (single-qubit two-flavor circuit, exact statevector validation, noise-type comparison, real IBM hardware) sits within this literature but has specific points of novelty, detailed below.

### Key papers identified

**1. Yeter-Aydeniz, K. & Siopsis, G. (2019).** *Neutrino oscillations in a quantum processor.* Phys. Rev. Research 1, 033176. arXiv:1904.10559.
Foundational paper for this project's approach. Demonstrated quantum simulation of both two-flavor and three-flavor neutrino oscillations, with the two-flavor case realized in a single qubit's 2D Hilbert space using individual single-qubit gates, and the three-flavor case requiring two entangled qubits (4D Hilbert space) with six rotation gates and two CNOTs implementing the 3×3 PMNS matrix. This paper directly informed this project's three-flavor extension section (Notebook 03) and is the primary citation for the two-flavor→three-flavor qubit-count reasoning used there.

**2. [2025 paper, Physica Scripta / IOP].** *Quantum circuits for simulating neutrino propagation in matter.* Phys. Scr. 100 (2025) 085111.
Closely related methodology: two-flavor oscillation on a single qubit with flavor states mapped to computational basis states, validated using both Qiskit AerSimulator and real IBM Quantum hardware against theoretical density-profile calculations. Most directly comparable prior work to this project's Notebooks 01-02 approach (exact simulator + real hardware validation), but focused on matter-propagation effects rather than the noise-robustness question this project investigates.

**3. Nguyen, H. C. et al. (2022).** *Simulating neutrino oscillations on a superconducting qutrit.* arXiv:2212.14170.
Alternative encoding: uses a single 3-level qutrit instead of multiple qubits to represent three flavors directly, validated on real hardware across vacuum, matter-interaction, and CP-violation scenarios. Cited in this project's three-flavor extension section as the alternative to the 2-qubit approach, noting qutrit hardware is less generally available than standard superconducting qubits.

**4. [2026 paper, arXiv:2606.18755].** *Quantum simulation of neutrino oscillations with bosonic encoding.*
Notes that two- and three-flavor simulations typically use one- and two-qubit encodings respectively (consistent with Finding/Notebook 03's three-flavor section), while exploring a third alternative: bosonic/cavity-mode (Fock-basis) encoding via SNAP and displacement gates. Useful citation demonstrating the field has multiple active encoding approaches beyond the qubit/qutrit dichotomy.

**5. [2025 paper, arXiv:2512.13143].** *Quantum critical dynamics and emergent universality in decoherent digital quantum processors.*
Not neutrino-specific, but methodologically important: uses the exact same Sampler-based error suppression options as this project (`twirling.enable_gates`, `twirling.enable_measure`, `dynamical_decoupling.enable`, all True), explicitly chosen as "minimal cost" mitigation for studying inherent hardware noise — directly supports this project's choice of mitigation technique (Finding 12) as a recognized, standard approach rather than an improvised workaround.

**6. [2026 paper, arXiv:2604.07452].** *Quantum Simulation of Collective Neutrino Oscillations using Dicke States.*
Uses `resilience_level=2` with Estimator primitives (dynamical decoupling + ZNE + Pauli twirling together) — useful contrast citation explaining why this project used the Sampler-native approach instead (resilience_level is Estimator-only in the current Qiskit runtime version, as discovered directly during this project's Finding 12 debugging). Also directly relevant to this project's "why quantum computing is needed" section, as collective/many-body neutrino oscillations are exactly the classically-intractable regime discussed there.

### Positioning paragraph (for paper Introduction/Discussion)

*"Quantum simulation of neutrino oscillation has been demonstrated using several encoding schemes: single- and two-qubit approaches (Yeter-Aydeniz & Siopsis 2019), qutrit-based encoding (Nguyen et al. 2022), and bosonic/cavity-mode encoding (2026). These works establish that quantum circuits can accurately reproduce oscillation probabilities, validated against theory and, in several cases, real quantum hardware. This project adopts the single-qubit two-flavor approach (consistent with Yeter-Aydeniz & Siopsis 2019 and the 2025 matter-propagation study), but extends beyond demonstration to ask a quantitative robustness question largely unaddressed in this literature: at what combined noise level does the circuit stop reliably recovering the oscillation pattern, and by what margin does this threshold exceed real current-hardware noise levels? This project further reports a direct, verified comparison of raw versus error-mitigated real-hardware execution (Finding 12), including an honest account of methodological pitfalls encountered (uncontrolled backend selection, job resubmission risk) — a level of process transparency not typically emphasized in prior demonstration-focused work, but valuable for reproducibility."*

### Reference list (for paper, formatted — fully verified)

1. Yeter-Aydeniz, K., & Siopsis, G. (2019). Neutrino oscillations in a quantum processor. *Physical Review Research*, 1(3), 033176. arXiv:1904.10559.
2. Joshi, S., Rajpoot, G., & Shukla, P. (2025). Quantum circuits for simulating neutrino propagation in matter. *Physica Scripta*, 100, 085111. arXiv:2503.20238. [Bhabha Atomic Research Centre, Mumbai, India.]
3. Nguyen, H. C., et al. (2022). Simulating neutrino oscillations on a superconducting qutrit. arXiv:2212.14170.
4. Joshi, S. (2026). Quantum simulation of neutrino oscillations with bosonic encoding. arXiv:2606.18755. [Bhabha Atomic Research Centre — same lead author as reference 2, extending the group's earlier qubit-based work to bosonic/cavity-mode encoding.]
5. Turro, F., Chernyshev, I. A., Bhaskar, R., & Illa, M. (2025). Qutrit and qubit circuits for three-flavor collective neutrino oscillations. *Physical Review D*, 111, 043038. arXiv:2407.13914.
6. Singh, G., Arvind, & Dorai, K. (2025). Simulating three-flavor neutrino oscillations on a nuclear magnetic resonance quantum processor. *Physica Scripta*, 100, 085106. arXiv:2412.15617.
7. Spagnoli, L., et al. (2025). Collective neutrino oscillations in three flavors on qubit and qutrit processors. *Physical Review D*, 111, 103054. arXiv:2503.00607.
8. Hall, B., Roggero, A., Baroni, A., & Carlson, J. (2021). Simulation of collective neutrino oscillations on a quantum computer. *Physical Review D*, 104, 063009. arXiv:2102.12556.
9. Bleau, K., Ilic, N., Kopp, J., Rahaman, U., & Yu, X. Y. (2026). Quantum Simulation of Collective Neutrino Oscillations using Dicke States. arXiv:2604.07452. [Johannes Gutenberg University Mainz / University of Toronto / CERN / TIFR Mumbai.]
10. Esteban, I., Gonzalez-Garcia, M. C., Maltoni, M., Schwetz, T., & Zhou, A. (2024). NuFit 6.0: updated global analysis of three-flavor neutrino oscillations. *JHEP*, 12, 216. nu-fit.org. [Source of θ₂₃ and Δm²₃₁ parameters used throughout this project.]

**Methodology-support citation (error mitigation technique justification, not neutrino-specific):**

11. Rhyno, B., Majumder, S., Vishveshwara, S., & Najafi, K. (2025). Quantum critical dynamics and emergent universality in decoherent digital quantum processors. arXiv:2512.13143. [University of Illinois Urbana-Champaign / IBM Quantum / MIT-IBM Watson AI Lab. Uses identical Sampler mitigation options — twirling.enable_gates, dynamical_decoupling.enable — as this project's Finding 12, described as "minimal cost" mitigation for studying inherent hardware noise.]

All author names and affiliations verified directly from arXiv listings. Reference list is submission-ready.

### Updated positioning paragraph (final, using fully verified references)

*"Quantum simulation of neutrino oscillation has been demonstrated using several approaches: single- and two-qubit encodings (Yeter-Aydeniz & Siopsis, 2019; Joshi et al., 2025), qutrit-based encoding (Nguyen et al., 2022; Turro et al., 2025), nuclear magnetic resonance processors (Singh et al., 2025), and bosonic cavity-mode encoding (Joshi, 2026). Collective, many-body neutrino oscillations — the classically-intractable regime motivating quantum simulation in the first place — have been explored by Hall et al. (2021), Spagnoli et al. (2025), and Bleau et al. (2026), among others. This project adopts the single-qubit two-flavor approach for its verifiability against exact theory, but extends beyond demonstration to address a quantitative robustness question not emphasized in this literature: at what combined noise level does the circuit stop reliably recovering the oscillation pattern, and by what safety margin does this exceed current real-hardware noise levels (Finding 11)? This project further reports a direct, verified comparison of raw versus error-mitigated real-hardware execution using standard Sampler-native techniques (dynamical decoupling, gate twirling), consistent with the minimal-cost mitigation approach used in recent decoherence studies (Rhyno et al., 2025), including a transparent account of methodological pitfalls encountered during real-hardware experimentation (Finding 12) — a level of process detail valuable for reproducibility but uncommon in demonstration-focused prior work."*

**Finding 13 — Statistical rigor pass (response to third review round)**

A third external review requested statistical rigor on two headline claims: the noise-threshold margin and the hardware mitigation improvement. Both were re-examined with repeated trials.

**Threshold, now with uncertainty:** The 50% contrast-retention threshold was recomputed across 5 seeds individually (rather than one seed as before): 1.174, 1.130, 1.135, 1.130, 1.145 → **mean 1.143 ± 0.017**. This is a tight spread, confirming the threshold is a robust, stable feature of the circuit/noise model rather than a single-seed artifact. Sensitivity to the failure-cutoff choice was also tested: 40% cutoff → severity 1.487; 50% → 1.142; 60% → 0.855. The corresponding robustness margin (vs. realistic noise at severity ≈0.03) therefore ranges from ≈28× (60% cutoff, most conservative) to ≈50× (40% cutoff), centered at ≈38× for the 50% cutoff — consistent with, and now properly bounding, the previously reported "~40×" figure.

**Hardware mitigation — CONCLUSION REVERSED.** The single-trial result reported in Finding 12 (raw MAE 0.0099, mitigated MAE 0.0095, +3.7% improvement) was re-tested with 3 independent repeated trials each (10 L/E points, 2000 shots, same pinned backend ibm_fez):
- Raw MAE across trials: 0.0121, 0.0122, 0.0152 → mean 0.0132 ± 0.0014
- Mitigated MAE across trials: 0.0156, 0.0153, 0.0113 → mean 0.0141 ± 0.0020
- Paired t-test: t = −0.524, p = 0.628 — **not statistically significant**

The mean mitigated MAE is now slightly *higher* than raw (opposite direction from Finding 12), but the difference is well within noise given the trial-to-trial spread. **Corrected conclusion: dynamical decoupling and gate twirling show no statistically significant effect, positive or negative, on this circuit's real-hardware accuracy.** The originally reported +3.7% improvement was a single favorable draw from a noisy system, not a reliable effect — precisely the failure mode the reviewer's skepticism anticipated (given the already-documented ~25% backend-selection confound in an earlier, discarded trial, single-trial comparisons on this hardware are not reliable).

This reversal is reported directly and is treated as a genuine, informative finding: it demonstrates that for a circuit this shallow, mitigation technique choice must be validated with repeated trials before any effect can be claimed, and Sampler-native DD/twirling mitigation in particular cannot be assumed beneficial without such validation.

---

## Open / Not Yet Done
- Rebuild remaining Phase 2 work (noise comparison, shot study) under corrected parameters, in notebook 03
- Real IBM Quantum hardware run (paused — resuming after Phase 1/2 rebuild is solid)
- Three-flavor extension (optional, only after main project complete)
- Paper draft (not started — Phase 3, per plan)
- AI assistance log (separate from this research log — needs to track prompts/errors/corrections specifically, per reviewer's request)

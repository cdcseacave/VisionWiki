---
title: "Relative Pose Estimation through Affine Corrections of Monocular Depth Priors (MADPose)"
type: paper
tags: [relative-pose, pose-estimation, monocular-depth, minimal-solver, lo-msac, ransac, cvpr-2025, highlight]
created: 2026-04-21
updated: 2026-04-21
sources: [papers/pataki2025_mp-sfm.md]
local_paper: papers/pose-estimation/yu_2025_madpose.pdf
url: https://arxiv.org/abs/2501.05446
code: https://github.com/MarkYu98/madpose
license_paper: arxiv-nonexclusive
license_code: BSD-3-Clause
status: draft
---

📄 [Full paper](../../papers/pose-estimation/yu_2025_madpose.pdf) · [arXiv](https://arxiv.org/abs/2501.05446) · [code](https://github.com/MarkYu98/madpose)

_Paper license: `arxiv-nonexclusive` · Code license: `BSD-3-Clause`_

## TL;DR
Relative-pose estimator that consumes **pixel matches + monocular depth priors** and explicitly models the per-view affine ambiguity `D̂ᵢ = aᵢDᵢ + bᵢ` of off-the-shelf MDE output. Derives three new minimal solvers (calibrated 3-point, shared-focal 4-point, two-focal 4-point) that jointly solve for pose, scale `α=a₂/a₁`, per-view shifts `βᵢ=bᵢ/aᵢ`, and focal lengths. Wraps them in a hybrid LO-MSAC that alternates depth-aware and classical point-based solvers and combines depth-induced reprojection error with Sampson epipolar error. CVPR 2025 Highlight.

## Problem
Classical relative pose (5pt / 6pt / 7pt) sees only epipolar geometry — it is pose-up-to-scale and blind to metric structure. Injecting monocular depth should in principle constrain the solution more tightly, but prior depth-aware solvers (Barath 2022 2pt+D, Ding 2024 3p3d / 4p4d) model depth as **scale-invariant** (α only). State-of-the-art MDE models are trained with **scale-and-shift-invariant** loss (α, β per view) or produce metric depth with empirically non-trivial per-image bias (Fig. 6: fitted β routinely > 10% of median depth even for metric-trained DA). The unmodeled shift is precisely what prevents depth priors from helping classical pose estimation.

## Method
Given `M` pixel correspondences `(p_{1j}, p_{2j})` with per-view mono-depth samples `d_{ij}=D_i(p_{ij})`, lift each 2D point to 3D as `P_{ij} = K_i^{-1} [p_{ij}, 1]^\top \cdot d̂_{ij}`, where `d̂_{1j} = d_{1j} + β_1` and `d̂_{2j} = α(d_{2j} + β_2)`. Since `(R, t)` is rigid, pairwise Euclidean distances on lifted 3D points must be equal across views:

$$\|P_{1j} - P_{1k}\|^2 = \|P_{2j} - P_{2k}\|^2$$

Expanding eliminates `R, t` entirely, leaving polynomial equations over `(α, β_1, β_2)` only.

**Calibrated 3-point solver.** `M=3` correspondences → `C(3,2) = 3` equations in `(α, β_1, β_2)`. Reparametrize `γ = α²` to reduce order, apply the Larsson 2017 Gröbner-basis template: 12×12 linear elimination + 4×4 eigenvalue → **up to 4 solutions**.

**Shared-focal 4-point solver.** Add `f` as fourth unknown, reparametrize `ω = 1/f²`, drop 4 of 6 equations for a minimal system → 36×36 template → **up to 8 solutions**.

**Two-focal 4-point solver.** Independent `f_1, f_2`, reparametrize `ω_i = 1/f_i²`, 5 equations from 6 → 40×40 linear elim + 4×4 eigenvalue → **up to 4 solutions**.

Recover `(R, t)` from each real, positive-depth solution by orthogonal Procrustes (SVD) on the back-projected 3D points.

**Hybrid LO-MSAC.** Inside a LO-MSAC loop (Lebeda 2012), alternate:

- Depth-aware solver (calibrated 3pt / shared-focal 4pt / two-focal 4pt) — sampled with probability learned from inlier-type ratios.
- Classical point-based solver (Nistér 5pt / Stewénius 6pt / 7pt fundamental) — fallback when depth priors are unreliable.

Each correspondence gets three scores: depth-induced reprojection `E_{r(1→2)}`, `E_{r(2→1)}`, and Sampson `E_s`. Combined MSAC score per correspondence is `min(E_r, τ_r) + min(min(E_s, τ_s), τ_r) · 2λ_s · τ_r / τ_s`, with `λ_s` empirically set to 1. Local optimization fits all shared parameters `(R, t, α, β_1, β_2, f, …)` jointly by least-squares over the current inlier partition. Implemented in C++ on top of RansacLib with pybind11 bindings; typical solve time 31–129 ms per pair on CPU.

## Results
Reported across ScanNet-1500 (indoor), MegaDepth-1500 (outdoor), ETH3D indoor, and sampled Stanford 2D-3D-S pairs. Best MDE: MoGe (calibrated), DA-met v2 (shared-focal / two-focal), Marigold (diversity). Best matchers: SP+LightGlue (sparse), RoMa (dense), MASt3R.

**ScanNet-1500 calibrated (Table 1)**:
| Matches | Method | MDE | AUC@10° ↑ |
|---|---|---|---|
| SP+LG | PoseLib-5pt | — | 39.11 |
| SP+LG | PoseLib-PnP | DA-met | 34.16 |
| SP+LG | 2pt+D&5pt (Barath 2022) | DA-met | 38.45 |
| SP+LG | **Ours-calib** | MoGe | **42.18** |

**ScanNet-1500 shared-focal (Table 3)**:
| Matches | Method | MDE | AUC@10° ↑ |
|---|---|---|---|
| MASt3R | PoseLib-6pt | — | 30.27 |
| MASt3R | 3p3d (Ding 2024) | DA-v2 | 52.28 |
| MASt3R | **Ours-sf** | DA-met | **56.99** |

**MegaDepth-1500 uncalibrated (Table 5)**:
| Method | MDE | AUC@10° ↑ |
|---|---|---|
| PoseLib-7pt | — | 36.80 |
| 4p4d (Ding 2024) | DA-v2 | 26.70 |
| **Ours-tf** | DA-met | **61.79** |

**Ablation (Table 6, calibrated, Marigold MDE)**: shift modeling gains +15 AUC@20° points on top of scale-only. **Ablation (Table 7)**: hybrid scoring is the single largest contributor among {hybrid solver, hybrid LO, hybrid score}; all three combined give +4–6 AUC@10° points.

## Why it matters
Three properties lift this beyond "another pose-from-depth paper":

1. **Consistent gains across every axis**: matcher (SP+LG / RoMa / MASt3R), MDE (DA-v1 / DA-v2 / Marigold / MoGe / metric DA), calibration regime (calibrated / shared-focal / two-focal), domain (indoor / outdoor). The gains compound with better matchers and better MDE models — no ceiling visible.
2. **Shift modeling is load-bearing empirically**: Fig. 6 shows that even metric-trained DA predicts depths whose fit-to-GT shift is typically > 10% of median scene depth. This is not a theoretical curiosity; it's the reason scale-only depth-aware solvers (2pt+D, 3p3d) previously underperformed or even regressed vs. point-only baselines in outdoor regimes. Explicit β modeling is the hinge.
3. **Pipeline-level decoupling**: the hybrid estimator degrades gracefully. When the depth prior is reliable, the depth-aware solver + depth-induced reprojection dominate. When it isn't (outdoor, foliage, reflections), the point-based solver + Sampson scoring take over. This is why the method is robust across domains without per-dataset tuning.

For the wiki, MADPose is the first **pose-level** consumer of monocular depth — previous ingested papers ([[pataki2025_mp-sfm]], [[zhong2026_instantsfm]]) consume mono-depth inside multi-view BA. This opens a new direction: depth-aware *two-view* pose as a standalone problem, with its own solver ecosystem and benchmark suite.

## Pipeline contribution

- [[affine-corrected-minimal-relative-pose-solvers_yu2025]] — `scope: stage-swap` on new stage [[pose-estimation.relative-pose-solver]]. Candidate thread: [[relative-pose-estimation]] op:default + [[mono-depth-estimation]] (as a new downstream consumer) · reusable for any RANSAC / MSAC loop consuming depth priors · expected gain: +15 AUC@20° vs scale-only (Table 6); solvers stand alone as minimal samplers.
- [[depth-induced-reprojection-scoring_yu2025]] — `scope: drop-in` on new stage [[pose-estimation.robust-estimator-scoring]]. Candidate thread: [[relative-pose-estimation]] · reusable as a scoring channel in any RANSAC that already computes affine-corrected depths · expected gain: single largest contributor to hybrid gains in Table 7 ablation.
- [[hybrid-lo-msac-dual-modality-estimator_yu2025]] — `scope: stage-swap` on new stage [[pose-estimation.hybrid-robust-estimator]]. Bundle node: `co_requires:` the two ideas above. Candidate thread: [[relative-pose-estimation]] op:default · expected gain: ScanNet-1500 shared-focal w/ MASt3R + DA-met 52.28 vs 30.27 baseline (+22 AUC@10°).
- **Synthesis-bet candidate** (cross-thread): MADPose's calibrated 3pt solver replaces [[mono-depth-normal-constrained-incremental-sfm_pataki2025]]'s median-ratio mono-depth rescaling (MP-SfM Eq. 1) in two-view bootstrap — upgrades MP-SfM's α-only alignment to α + β₁ + β₂ jointly-solved alignment. Flagged as Bet #014 below.
- **Synthesis-bet candidate**: depth-induced reprojection as a third model inside [[sfm.geometric-verification]]'s multi-model RANSAC (currently H + F) — catches scenes where epipolar passes but metric depths disagree. Flagged below.

## Relation to prior work
- Directly extends **Barath et al. 2022** ([4] in refs) 2pt+D: they solve pose under the scale-only depth assumption, which is rank-deficient for rigid alignment. MADPose adds the shift parameters that make the system well-posed.
- Directly extends **Ding et al. 2024** ([17]) 3p3d / 4p4d depth-aware fundamental matrix solvers: same augmentation (add β₁, β₂), reformulate via length-preservation.
- Orthogonal to **DUSt3R / MASt3R** ([[mast3r|MASt3R]]) feed-forward pair pose: Table 5 shows MADPose using MASt3R matches beats pure MASt3R by +7 AUC@10° on MegaDepth uncalibrated — the classical solver pipeline remains competitive when paired with strong learned matches and depth.
- Orthogonal to [[mono-depth-normal-constrained-incremental-sfm_pataki2025]] MP-SfM: both use mono-depth priors, but MP-SfM is multi-view incremental SfM (PnP + BA), MADPose is pair-pose. They solve different stages.
- Uses **Depth-Anything v1/v2** (Yang 2024) and [[wang2025_moge|MoGe]] as MDE backbones.
- Uses **PoseLib** / **RansacLib** solver + RANSAC infrastructure; depends on **Ceres Solver** for local optimization (auto-differentiated Jacobians).

## Open questions / limitations
- **Pair-pose only**: the hybrid estimator doesn't propagate shift estimates across an image sequence. Two pairs sharing view `I_k` can disagree on `β_k` — unresolved. For SfM integration this is a known gap.
- **Degenerate cases**: 3-point samples on near-coplanar scene geometry degrade solver conditioning; paper notes but does not quantify the failure rate.
- **Runtime**: 31–129 ms per pair is fast for RANSAC but not real-time SLAM budget. Authors note analytic Jacobians + GPU BA would help.
- **My skepticism**: the hybrid estimator's solver-selection probabilities are learned online per image pair — behaviour on adversarial scenes (depth prior confident but wrong, epipolar ambiguous) is not analyzed. The graceful-degradation story is empirically strong but not theoretically characterized.
- **Not tested**: integration with incremental SfM (does replacing COLMAP's 5pt init with MADPose improve downstream reconstruction?), with feed-forward pointmap models (DUSt3R/MASt3R predict depth natively — is there a principled way to fuse their depth into the MADPose estimator vs. using off-the-shelf DA/MoGe?).

## Code & license
BSD-3-Clause on code, standard arxiv-nonexclusive on paper. No commercial-use blockers. Depends on PoseLib (BSD), RansacLib (BSD), Ceres Solver (BSD) — full stack commercial-OK.

## References added to the wiki
- [[affine-corrected-minimal-relative-pose-solvers_yu2025]] (new idea)
- [[depth-induced-reprojection-scoring_yu2025]] (new idea)
- [[hybrid-lo-msac-dual-modality-estimator_yu2025]] (new idea)
- [[pose-estimation.relative-pose-solver]] (new stage)
- [[pose-estimation.robust-estimator-scoring]] (new stage)
- [[pose-estimation.hybrid-robust-estimator]] (new stage)
- [[wang2025_moge|MoGe]] (new stub)
- [[relative-pose-estimation]] (new thread)

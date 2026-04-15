---
title: "Tanks and Temples: Benchmarking Large-Scale Scene Reconstruction"
type: paper
tags: [benchmark, dataset, multi-view-stereo, sfm, 3d-reconstruction, ground-truth, laser-scan]
created: 2026-04-15
updated: 2026-04-15
sources: [papers/schonberger2016_colmap-sfm.md, papers/schonberger2016_colmap-mvs.md]
local_paper: papers/datasets-benchmarks/knapitsch_2017_tanks-and-temples.pdf
url: https://dl.acm.org/doi/10.1145/3072959.3073599
status: stable
---

📄 [Full paper](../../papers/datasets-benchmarks/knapitsch_2017_tanks-and-temples.pdf) · [ACM SIGGRAPH 2017](https://dl.acm.org/doi/10.1145/3072959.3073599) · [Project](https://www.tanksandtemples.org/)

## TL;DR

Knapitsch, Park, Zhou & Koltun (SIGGRAPH 2017, Intel Labs) introduced the **Tanks and Temples** benchmark: 14 large-scale real-world scenes captured with high-res video, paired with **submillimeter industrial laser-scanner ground truth** and a precision/recall **F-score** evaluation protocol. The dataset is split into two groups (*Intermediate*, *Advanced*) and became the single most-used large-scene reconstruction benchmark for the decade that followed — cited by essentially every MVS, NeRF, and 3DGS paper evaluating geometry quality. See [[tanks-and-temples]] for the dataset page.

## Problem

Before 2017, large-scale reconstruction had no usable benchmark:
- **DTU** and the Middlebury stereo sets covered lab-sized objects with controlled illumination and known poses — they eliminated exactly the failure modes large-scale pipelines struggled with.
- **Strecha 2008** and **Merrell 2007** were outdoor but limited in scene variety and didn't push modern pipelines to failure.

The community could iterate on ideas but had no shared target that exercised **full pipelines** (camera tracking + dense reconstruction) on scenes that actually broke them.

## Method (of the benchmark itself)

### Data acquisition
- **Ground truth**: a laser scanner with 330 m range and submillimeter accuracy captures a point cloud of each scene. Multiple scans per scene are registered into a unified ground-truth model.
- **Input modality**: 8-megapixel video — specifically *video*, not still-image bursts, so pipelines can exploit temporal coherence (frame-to-frame tracking, rolling-shutter correction, motion blur reasoning).

### Scene splits (14 scenes)
- **Training** (7 scenes): Barn, Caterpillar, Church, Courthouse, Ignatius, Meetingroom, Truck. Ground truth public.
- **Intermediate test** (8 scenes): Family, Francis, Horse, Lighthouse, M60, Panther, Playground, Train. Ground truth withheld; evaluation via online server.
- **Advanced test** (6 scenes): Auditorium, Ballroom, Courtroom, Museum, Palace, Temple. Harder — larger, more reflective, more texture-starved.

### Evaluation metric — F-score
For each scene, a per-scene distance threshold $\tau$ is set from ground-truth nearest-neighbor statistics. Given a reconstructed point set $R$ and ground-truth $G$:
- **Precision** = fraction of $R$ with nearest GT point within $\tau$
- **Recall** = fraction of $G$ with nearest reconstructed point within $\tau$
- **F-score** = $2 \cdot \frac{P \cdot R}{P + R}$

The dual precision+recall framing (rather than just Chamfer distance) is the key methodological choice — it separates "reconstruction that is accurate where it exists" from "reconstruction that covers the scene at all." Modern methods routinely have high precision and mediocre recall, or vice versa; the F-score surfaces this honestly.

## Results (as reported in the paper — state-of-the-art in 2017)

The paper reports F-scores for the major pipelines of the era — COLMAP, MVE, OpenMVG+OpenMVS, PMVS, Pix4D, Alice Vision. COLMAP dominates at the top but no pipeline is uniformly best; Advanced-split scenes remain unsolved at publication (F-scores mostly below 20–30%).

These numbers are primarily historical now; what matters is that the *framework* survived — every subsequent paper reports T&T F-scores using the same protocol.

## Pipeline contribution

Benchmark papers don't contribute pipeline *components* but they define the *success criteria* the pipelines are tuned to. Concrete contributions to the wiki's threads:

- **F-score evaluation protocol (precision + recall, per-scene $\tau$)** — the metric that every [[gaussian-to-mesh-pipelines]] and [[radiance-field-evolution]] paper's "Current SOTA pipeline" is optimized against. Without precision + recall, "better" collapses to Chamfer and the thin-but-accurate degenerate optimum is rewarded.
- **Intermediate vs. Advanced split** — defines two different "better" targets. Intermediate split is saturating (F>0.5 common); Advanced split remains unsolved and is where synthesis bets should be aimed.
- **Submillimeter GT precision** → the benchmark won't saturate from GT noise. Synthesis bets that claim +X F-score on Advanced are measurable and credible.
- **Successor candidates**: DTU (smaller, lab), ETH3D (smaller outdoor), ScanNet (indoor RGB-D). For [[gaussian-to-mesh-pipelines]] the current SOTA-pipeline goal statements should name T&T (Intermediate + Advanced) + DTU as the dual benchmarks; neither alone is sufficient.

## Why it matters

Tanks and Temples is **the reason we can compare reconstruction papers across a decade**. Its longevity comes from three design choices that have aged remarkably well:

1. **Submillimeter laser-scanner ground truth** — no NeRF/3DGS method has yet saturated the metric because the GT is effectively noiseless relative to reconstruction error.
2. **F-score rather than single-number Chamfer** — forces papers to report completeness *and* accuracy; prevents the "thin-but-accurate" degenerate optimum.
3. **Video inputs with withheld GT** — means submissions are evaluated on held-out scenes, not overfit to visible-GT data.

Even papers that measure themselves on other datasets (DTU, ETH3D, ScanNet) typically also report T&T as the large-scale sanity check.

## Relation to prior work

- Successor to DTU (Aanæs et al. 2016) and Strecha 2008, but deliberately harder and larger.
- Orthogonal to ScanNet (Dai et al. 2017 — indoor, RGB-D, different metric focus) and ETH3D (Schöps et al. 2017 — smaller outdoor scenes, different GT mechanism).
- Reported results from COLMAP ([[schonberger2016_colmap-sfm|Schönberger & Frahm 2016]], [[schonberger2016_colmap-mvs|Schönberger et al. 2016]]) positioned it as the classical baseline to beat.

## Open questions / limitations

- **No semantic / material GT** — only geometry. Papers that care about appearance or materials have to bring their own ground truth.
- **Static scenes only** — no dynamic content, no motion; recent dynamic reconstruction methods (MegaSaM, MonST3R) can't use T&T for their target use case.
- **Registration error** in the laser scans themselves, while submillimeter locally, can reach centimeter-scale when multiple scans are merged over large areas — mostly below reconstruction error but occasionally surfaces as a floor.
- **Benchmark saturation** — the Intermediate split is increasingly close to saturated by modern 3DGS/NeRF methods (F-scores >0.5 are now routine where 2017 SOTA was ~0.3). The Advanced split still has room.

## References added to the wiki

- [[tanks-and-temples]] — the dataset page
- [[multi-view-stereo]]
- [[structure-from-motion]]

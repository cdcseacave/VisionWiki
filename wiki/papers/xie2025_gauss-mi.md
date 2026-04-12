---
title: "GauSS-MI: Gaussian Splatting Shannon Mutual Information for Active 3D Reconstruction"
type: paper
tags: [3d-gaussian-splatting, active-reconstruction, uncertainty-quantification, mutual-information, next-best-view, robotics]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/radiance-fields/xie_2025_gauss-mi.pdf
url: https://arxiv.org/abs/2504.21067
status: draft
---

📄 [Full paper](../../papers/radiance-fields/xie_2025_gauss-mi.pdf) · [arXiv](https://arxiv.org/abs/2504.21067)

## TL;DR

GauSS-MI introduces a probabilistic model that quantifies visual uncertainty for each Gaussian primitive in a [[3d-gaussian-splatting]] map, then leverages [[Shannon-mutual-information]] to formulate a real-time criterion for selecting the next best view during active 3D reconstruction. The method is integrated into a full active reconstruction system with a view and motion planner, achieving superior visual fidelity and reconstruction efficiency compared to prior exploration-based and information-theoretic approaches.

## Problem

Existing active 3D reconstruction methods lack a principled way to evaluate visual uncertainty in real time within radiance-field-based representations. Prior information-theoretic approaches like FisherRF use the Fisher information matrix for next-best-view (NBV) selection but neglect the real-time demands of active reconstruction. Exploration-based planners (e.g., FUEL) do not account for visual quality at all, leading to inefficient reconstruction paths.

## Method

1. **Probabilistic model for 3DGS**: Each Gaussian's color (via [[spherical-harmonics]] coefficients) is modeled as a random variable with learned mean and variance, enabling per-Gaussian visual uncertainty quantification.
2. **GauSS-MI criterion**: Using Shannon Mutual Information, the expected information gain from an arbitrary novel viewpoint is computed analytically by propagating per-Gaussian uncertainty through the [[differentiable-rendering]] pipeline. This avoids costly sampling or ray marching.
3. **Active reconstruction system**: GauSS-MI is integrated with a view and motion planner that evaluates candidate viewpoints in real time and selects the next best view that maximizes expected information gain, then plans an optimal motion primitive to reach it.

## Results

Evaluated on three simulation scenes (Oil Drum, Drilling Machine, Potted Plant) with comparisons against FUEL and NARUTO:

- **Oil Drum**: GauSS-MI achieves PSNR 34.35 / SSIM 0.986 / LPIPS 0.068 with only 141 frames, vs. FUEL's 22.82 PSNR with 165 frames and NARUTO's 31.84 PSNR with 3000 frames.
- **Drilling Machine**: PSNR 33.99 / SSIM 0.995 / LPIPS 0.040 with 122 frames.
- GauSS-MI consistently achieves the highest efficiency metric E = PSNR / log(N_f), demonstrating better visual quality with shorter reconstruction paths.
- Real-world experiments on a robotic arm platform further validate the system.

## Why it matters

This work bridges information theory and Gaussian splatting for robotics applications. By providing a closed-form, real-time uncertainty metric for [[3d-gaussian-splatting]], it enables autonomous systems to make intelligent decisions about where to look next, which is critical for robotic exploration, inspection, and digital twin creation.

## Relation to prior work

- Builds on [[3d-gaussian-splatting]] (Kerbl et al., 2023) as the underlying scene representation.
- Extends information-theoretic active perception from occupancy grids (CSQMI, FSMI) to radiance fields.
- Contrasts with FisherRF, which uses the [[Fisher-information-matrix]] but is not designed for real-time active reconstruction.
- Compared against exploration-based planners like FUEL and learning-based NARUTO.

## Open questions / limitations

- The probabilistic model assumes independent Gaussian color distributions; correlations between overlapping Gaussians are not modeled.
- Evaluated primarily in simulation and single-object tabletop settings; scalability to large outdoor scenes is unexplored.
- Does not address geometric uncertainty, only visual (color) uncertainty.
- The system currently requires known robot kinematics for motion planning.

## References added to the wiki

- [[3d-gaussian-splatting]]
- [[Shannon-mutual-information]]
- [[spherical-harmonics]]
- [[differentiable-rendering]]
- [[Fisher-information-matrix]]
- [[next-best-view]]

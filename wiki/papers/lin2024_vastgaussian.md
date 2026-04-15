---
title: "VastGaussian: Vast 3D Gaussians for Large Scene Reconstruction"
type: paper
tags: [radiance-fields, 3dgs, large-scale, scene-partitioning, appearance-modeling, aerial]
created: 2026-04-14
updated: 2026-04-15
sources:
  - papers/lin2024_vastgaussian.md
local_paper: papers/radiance-fields/lin_2024_vastgaussian.pdf
url: https://arxiv.org/abs/2402.17427
status: draft
---

📄 [Full paper](../../papers/radiance-fields/lin_2024_vastgaussian.pdf) · [arXiv](https://arxiv.org/abs/2402.17427) · [project page](https://vastgaussian.github.io)

*Lin, Li, Tang, Liu, Liu, Liu, Lu, Wu, Xu, Yan, Yang — CVPR 2024 (Tsinghua · Huawei Noah's Ark · CAS).*

## TL;DR

VastGaussian is the first method to scale [[3d-gaussian-splatting]] to aerial / city-scale scenes while keeping real-time rendering. It divides the scene into cells via a progressive partitioning strategy with an **airspace-aware visibility criterion**, trains each cell in parallel, then seamlessly merges. A **decoupled appearance module** absorbs exposure / white-balance drift across captures during training without inflating inference cost.

## Problem

Naïve 3DGS breaks on large scenes for three reasons:

1. **VRAM exhaustion.** A small Mip-NeRF 360 scene (~100 m²) already needs ~5.8 M Gaussians. Scaling to km² aerial captures requires tens of millions — a single 32 GB GPU OOMs or cannot run enough iterations.
2. **Undertrained detail.** Even when memory fits (by increasing the densification interval), each cell of the scene gets too few gradient updates → blurry textures, low PSNR.
3. **Appearance variation.** Aerial datasets span minutes / hours of flight: exposure, sun angle, white balance drift between images. 3DGS absorbs this as **floaters** — low-opacity blobs positioned to explain per-image brightness differences from specific viewpoints.

Prior NeRF-based large-scene methods ([Mega-NeRF](https://arxiv.org/abs/2112.10703), [Switch-NeRF](https://openreview.net/forum?id=PQ2zoIZqvm), [Grid-NeRF](https://arxiv.org/abs/2303.14001)) solve partitioning and appearance but render at ≤0.3 FPS and produce blurry novel views.

## Method

### Progressive data partitioning

The scene is divided into `m × n` cells on the ground plane so each cell holds a roughly balanced camera count. For each cell `j`, the training set is built in four stages (Fig. 3 of the paper):

**(a) Camera-position region division.** Partition ground-plane camera footprints into an `m × n` grid; subdivide rows to equalize counts.

**(b) Position-based data selection.** Expand each cell's rectangle outward by ~20% on all sides; assign any cameras/points inside the expanded rectangle to that cell. The overlap provides cross-cell context and is trimmed at merge time.

**(c) Visibility-based camera selection — airspace-aware.** Add cameras whose cell-visibility exceeds threshold $T_h = 25\%$. Visibility is defined as the projected area of the cell divided by image area $\Omega_{ij}^{\text{air}} / \Omega_i$. *Critically, the cell's 3D extent is an **axis-aligned bounding box from ground to the highest point**, not just the surface convex hull.* The airspace-agnostic version — projecting only surface points — under-counts visibility, lets floaters fill the sky, and misses distant cameras that observe the cell.

**(d) Coverage-based point selection.** Add all points visible to any camera selected in (c), even if those points lie outside the cell. This better initializes Gaussians inside the cell — cheap reusable supervision that prevents floater creation.

### Per-cell optimization + seamless merging

Each cell trains a standard 3DGS model independently on its camera / point subset (60 k iterations, densification 1 k–30 k with 1 k interval, all other hyperparameters matching Kerbl 2023). After training, Gaussians whose centers fall **outside the original un-expanded cell region** are discarded. Because cells share their overlap region during training, the remaining Gaussians stitch together seamlessly — no post-hoc blending or appearance re-alignment (as Block-NeRF needs).

### Decoupled appearance modeling

Rather than baking per-image appearance into the radiance model (NeRF-W, Ha-NeRF) — which requires an MLP at render time and is incompatible with splatting's rasterization — VastGaussian transforms only the **training-time target**.

The flow (Fig. 4):

1. Rendered image $\mathcal{I}_i^r$ is downsampled 32× and concatenated with a per-image **appearance embedding** $\ell_i$ (length 64), producing a feature map $\mathcal{D}_i$ of shape $\tfrac{H}{32} \times \tfrac{W}{32} \times 67$.
2. A small CNN (4 upsampling blocks with pixel-shuffle + 3×3 conv, then bilinear upsample to $H \times W \times 3$) produces a **transformation map** $\mathcal{M}_i$.
3. The adjusted image $\mathcal{I}_i^a = T(\mathcal{I}_i^r; \mathcal{M}_i)$ — a pixel-wise multiplication suffices in practice — is matched to the GT:

$$
\mathcal{L} = (1-\lambda)\mathcal{L}_1(\mathcal{I}_i^a, \mathcal{I}_i) + \lambda\, \mathcal{L}_{\text{D-SSIM}}(\mathcal{I}_i^r, \mathcal{I}_i)
$$

The L1 term (post-transform) learns the appearance delta; the D-SSIM term (pre-transform) forces the Gaussians to learn consistent *structure*. At inference, $\mathcal{M}_i$ and $\ell_i$ are discarded — no runtime overhead.

### Why 3DGS can't use NeRF-W's trick directly

NeRF-W / Block-NeRF concatenate the appearance embedding at the MLP's point-wise radiance query, so each ray samples an appearance-conditioned color. 3DGS has no MLP in the render path — Gaussians carry explicit SH coefficients and are rasterized directly. VastGaussian sidesteps this by moving the learned variation *out of the representation entirely* into a per-image 2D transform.

## Results

**Datasets.** Mill-19 (Rubble, Building) and UrbanScene3D (Campus, Residence, Sci-Art) — aerial captures, 4× downsampled, 1 in 4 held out.

**Headline numbers** (mean across 5 scenes, 1080p):

| Method | SSIM | PSNR | LPIPS | Training (UrbanScene3D) | VRAM | FPS |
|---|---|---|---|---|---|---|
| Mega-NeRF | 0.547 | 23.42 | 0.454 | 29h19m | 5.9 G | 0.23 |
| Switch-NeRF | 0.579 | 23.62 | 0.421 | 41h5m | 4.1 G | 0.19 |
| Modified 3DGS | 0.777 | 24.81 | 0.166 | 31h12m | 31.6 G | **210.99** |
| **VastGaussian** | **0.836** | **24.89** | **0.132** | **2h56m** | **11.9 G** | 171.92 |

Wins SSIM and LPIPS on **all 5 scenes** — dominance on LPIPS (perceptual quality) is the cleanest signal. Modified 3DGS renders faster than VastGaussian (fewer Gaussians post-densification starvation) but is visibly blurry with floaters (Fig. 1, Fig. 5). Cell count sweet spot is ~8 (Tab. 4); >16 cells shows diminishing returns and wastes VRAM.

**Ablations** (Sci-Art, Tab. 3):

- w/o visibility-based camera selection (VisCam): PSNR drops 25.08 → 20.05, heavy boundary appearance-jumping artifacts.
- w/o coverage-based point selection (CovPoint): 25.08 → 26.14 — *counterintuitively improves PSNR but adds airspace floaters*. CovPoint trades a small photometric score for visual cleanliness.
- airspace-agnostic visibility: 26.81 → 24.54, heavy floaters (Fig. 8).
- w/o decoupled appearance modeling: 26.81 → 25.08, floaters return to compensate for brightness drift.

## Why it matters

- **First real-time large-scene method in the 3DGS family.** Every subsequent city-scale / aerial 3DGS paper (Hierarchical 3DGS, CityGaussian, etc.) builds on the partition-and-merge pattern VastGaussian established.
- **Decoupled appearance modeling is the clean 3DGS answer** to the GLO/NeRF-W lineage — transforming targets not representation is a reusable trick for any rasterization-based neural-free renderer with photometric drift.
- **Airspace-aware visibility** is a subtle insight: the natural "surface convex hull" definition fails precisely because floaters live *off* the surface. Defining visibility over the full vertical airspace is what prevents the method from silently re-creating the floater problem it aims to fix.

## Relation to prior work

- Builds on [[3d-gaussian-splatting]] (Kerbl 2023) as the core representation.
- Inherits **scene partitioning + parallel cell training + merge** from [Block-NeRF](https://arxiv.org/abs/2202.05263) (Tancik 2022) and [Mega-NeRF](https://arxiv.org/abs/2112.10703) (Turki 2022) — but drops Block-NeRF's expensive post-hoc appearance alignment because the decoupled training module already learns consistent geometry.
- Decoupled appearance modeling is the 3DGS-compatible successor to **GLO appearance embeddings** (NeRF-W, Ha-NeRF) — the embedding still exists but acts on a 2D transform rather than the 3D radiance field.
- Contrasts with [EA-3DGS](guo2025_ea-3dgs.md) and [VPGS-SLAM](deng2026_vpgs-slam.md), which tackle large-scale 3DGS via **compression** (quantization, codebooks) and **incremental SLAM** respectively; VastGaussian's approach is **spatial partitioning** — orthogonal and combinable.

## Open questions / limitations

- **Partitioning requires a flat-ish ground plane.** The method aligns the world y-axis to ground via Manhattan alignment [1] and grids ground-plane camera positions. Non-planar scenes (canyons, multi-story indoor) would need a different partitioning primitive.
- **No automatic cell-count selection.** User picks `m × n`; the paper shows 8 is best for their aerial scenes but gives no method for choosing it from scene geometry or camera distribution.
- **Storage scales linearly with cell count.** 24 cells → 32.6 M Gaussians. For truly city-scale (km²) scenes, some form of LOD / compression (EA-3DGS style) would need to be layered on top.
- **Appearance module assumes pixel-wise corrections suffice.** Works on Mill-19 / UrbanScene3D, but scenes with strong view-dependent specular drift (wet surfaces, reflective glass at varying sun angles) may need the affine or γ-corrected variants in the supplement (Tab. 6 shows marginal gains there anyway).
- **The L1/D-SSIM split still lets appearance leak into geometry.** The D-SSIM term runs on the pre-transform render, but SSIM itself fuses luminance, contrast, and structure — so the appearance embedding can still mask geometric errors via contrast/structure compensation. [CoMe (Radl 2026)](radl2026_confidence-mesh-3dgs.md) later refines this by decoupling SSIM into $l \cdot c \cdot s$ and letting the embedding correct *only* luminance, freeing contrast/structure to supervise geometry directly.
- **No dynamic-object handling.** All floaters are assumed to come from appearance drift, not from moving cars/pedestrians. DroneSplat-style distractor masking would be complementary.

## Pipeline contribution

- **Airspace-aware visibility criterion for partitioning (N1)** — visibility defined over full ground-to-sky axis-aligned bounding box, not surface convex hull. candidate thread: [[radiance-field-evolution]] city-scale · stage: *cell construction for partitioned training* · replaces/augments: *surface-convex-hull visibility* · expected gain: prevents floaters from filling sky; PSNR delta 26.81 → 24.54 in airspace-agnostic ablation (visible 2+ dB regression when disabled).
- **Progressive 4-step partition (region div → position-based → visibility-based → coverage-based) (N2)** — builds per-cell training sets with principled overlap. candidate thread: [[radiance-field-evolution]] city-scale · stage: *data-partition pipeline* · replaces/augments: *uniform spatial cropping (Mega-NeRF)* · expected gain: seamless merge without post-hoc blending; 10× shorter training vs Mega-NeRF.
- **Decoupled appearance modeling via 2D transformation map (N3)** — per-image CNN transforms rendered target; appearance discarded at inference. candidate thread: [[radiance-field-evolution]] · stage: *appearance drift compensation* · replaces/augments: *NeRF-W GLO embeddings on the representation* · expected gain: 3DGS-compatible (rasterization-native), no runtime overhead, +1.7 dB PSNR on Sci-Art.
- **Synthesis-bet enabler**: VastGaussian's decoupled appearance module is explicitly combined with CoMe's SSIM decoupling in the downstream [[come-integration-nerfstudio]] design. The composition is a concrete novel-pipeline implementation of a Batch-1 synthesis pattern.

## Downstream in this wiki

- The decoupled appearance module is ported into the visiofacto/nerfstudio design at [[come-integration-nerfstudio]] (Part 7), combined with CoMe's SSIM decoupling so the per-image embedding compensates only for luminance.

## References added to the wiki

- Created: `wiki/papers/lin2024_vastgaussian.md` (this page).
- Updated: `wiki/methods/3d-gaussian-splatting.md` (large-scale variants section).
- Updated: `wiki/threads/radiance-field-evolution.md` (scaling section expanded with spatial-partitioning direction).
- Updated: `wiki/designs/come-integration-nerfstudio.md` (new Part 7 specifying the appearance module integration).

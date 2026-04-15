---
title: "Gaussian Splatting with Discretized SDF for Relightable Assets"
type: paper
tags: [3d-gaussian-splatting, signed-distance-field, inverse-rendering, relighting, BRDF, material-decomposition]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/zhu_2025_gs-discretized-sdf.pdf
url: https://arxiv.org/abs/2507.15629
status: draft
---

📄 [Full paper](../../papers/radiance-fields/zhu_2025_gs-discretized-sdf.pdf) · [arXiv](https://arxiv.org/abs/2507.15629)

## TL;DR

This paper introduces a discretized [[signed-distance-field]] (SDF) representation within [[3d-gaussian-splatting]] for inverse rendering. Instead of maintaining a separate continuous SDF network (which increases memory and training complexity), each Gaussian stores a sampled SDF value. An SDF-to-opacity transformation links the discrete SDF to Gaussian opacity, enabling SDF rendering via splatting. A projection-based consistency loss regularizes the discrete samples to match the underlying surface, yielding state-of-the-art relighting quality among Gaussian-based methods.

## Problem

Applying [[3d-gaussian-splatting]] to inverse rendering (decomposing geometry, material, and lighting) is challenging because the discrete nature of Gaussian primitives makes it difficult to enforce geometric constraints like the Eikonal loss. Recent methods add a continuous SDF as an extra representation to regularize geometry, but this increases memory usage and complicates training with dual representations.

## Method

1. **Discretized SDF**: Each Gaussian primitive stores a scalar SDF value, representing a discrete sample of the underlying continuous SDF. This avoids maintaining a separate neural SDF network.
2. **SDF-to-opacity transformation**: The stored SDF value is converted to Gaussian opacity via a differentiable transformation, linking the SDF surface (zero-level set) to the rendered opacity. A median loss encourages Gaussians to converge onto the opaque surface.
3. **Projection-based consistency loss**: Since gradient-based constraints (e.g., Eikonal loss) cannot be directly applied to discrete SDF samples, the method projects Gaussians onto the zero-level set of the SDF and enforces alignment between the splatting surface and the SDF surface.
4. **Spherical initialization**: Gaussians are initialized on a sphere to avoid local minima during early optimization.
5. **BRDF decomposition**: Standard PBR material model with base color, roughness, and metallic parameters, using [[spherical-harmonics]] for environment lighting.

## Results

Evaluated on Glossy Blender, NeRO, Ref-NeRF, and NeILF++ datasets for relighting quality:

- Outperforms GShader (mean PSNR 16.95), GS-IR (mean PSNR ~16), and other Gaussian-based inverse rendering methods on the Glossy Blender dataset.
- Produces cleaner normal estimation and more robust geometry/material decomposition.
- Training takes ~0.5h with 4GB memory, comparable to other GS-based methods.
- Achieves photo-realistic relighting on real scenes (e.g., NeILF++ Qilin scene).
- Ablation confirms each component (discretized SDF, projection loss, spherical init) consistently improves relighting PSNR/SSIM.

## Why it matters

This work provides an elegant solution to the geometry regularization problem in Gaussian-based inverse rendering. By encoding the SDF directly within each Gaussian primitive rather than maintaining a separate continuous field, it achieves the benefits of SDF-based geometry priors (robust decomposition) without the memory and complexity overhead. This advances the goal of creating relightable 3D assets from multi-view images using efficient Gaussian representations.

## Pipeline contribution

- **Per-Gaussian discretized SDF value (N1)** — each Gaussian stores a scalar SDF sample, no separate continuous SDF network. candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm A + inverse-rendering · stage: *per-primitive geometry representation* · replaces/augments: *dual continuous SDF + Gaussian networks (GS2Mesh, 3DGSR)* · expected gain: half the memory, simpler training loop, SOTA relighting on Glossy Blender.
- **SDF-to-opacity transformation with median loss (N2)** — differentiable mapping linking discrete SDF samples to Gaussian opacity; median loss pulls Gaussians onto the zero level set. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *SDF-to-opacity bridge* · replaces/augments: *ad-hoc opacity penalties* · expected gain: Gaussians converge on opaque surface; mesh + appearance from same representation.
- **Projection-based consistency loss (N3)** — projects Gaussians to zero-level-set, enforces alignment between splat surface and SDF surface. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *SDF-Gaussian consistency* · replaces/augments: *Eikonal loss (not applicable to discrete samples)* · expected gain: discrete-SDF analog of Eikonal, works at splat resolution.
- **Synthesis-bet**: *CoMe confidence as Zhu 2025's discrete-SDF confidence* — the two self-supervised signals (confidence + SDF) provide complementary geometry/appearance disentanglement.
- **Role**: adds the **relightable/inverse-rendering** lane to the thread — no pure-mesh-extraction paper does relighting; this is where Gaussian-to-mesh meets physical-rendering.

## Relation to prior work

- Builds on [[3d-gaussian-splatting]] (Kerbl et al., 2023) as the rendering primitive.
- Extends inverse rendering methods like GShader and [[GS-IR]] by introducing SDF regularization without a separate neural field.
- Related to [[NeuS]] which uses SDF with neural radiance fields, but adapted for the discrete Gaussian setting.
- Contrasts with GS2Mesh and 3DGSR which use separate continuous SDF networks alongside Gaussians.
- Uses [[spherical-harmonics]] for environment map representation.

## Open questions / limitations

- Only considers direct illumination for efficiency; objects with complex inter-reflections or occlusions may not be handled well. Adding indirect illumination is identified as future work.
- Mesh extraction with BRDF parameters is left for future work.
- Currently limited to bounded object-scale scenes; extending to unbounded scenes is a promising future direction.
- The discrete SDF approximation may not capture very fine geometric details compared to continuous SDF methods.

## References added to the wiki

- [[3d-gaussian-splatting]]
- [[signed-distance-field]]
- [[NeuS]]
- [[GS-IR]]
- [[spherical-harmonics]]
- [[inverse-rendering]]
- [[BRDF]]

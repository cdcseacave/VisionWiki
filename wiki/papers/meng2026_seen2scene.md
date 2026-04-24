---
title: "Seen2Scene: Completing Realistic 3D Scenes with Visibility-Guided Flow"
type: paper
tags: [scene-completion, generative-3d, flow-matching, tsdf, sparse-transformer, controlnet, visibility-aware, indoor-scene]
created: 2026-04-24
updated: 2026-04-24
sources: []
local_paper: papers/mesh-reconstruction/meng_2026_seen2scene.pdf
url: https://arxiv.org/abs/2603.28548
code: https://github.com/quan-meng/seen2scene
license_paper: arxiv-nonexclusive
license_code: unknown
status: stable
---

📄 [Full paper](../../papers/mesh-reconstruction/meng_2026_seen2scene.pdf) · [arXiv](https://arxiv.org/abs/2603.28548) · [project](https://quan-meng.github.io/projects/seen2scene/) · [code](https://github.com/quan-meng/seen2scene)

_Paper license: `arxiv-nonexclusive` · Code license: `unknown` (no LICENSE file in repo as of 2026-04-24)_

## TL;DR

Seen2Scene is the first flow-matching-based 3D scene-completion / generation model that **trains directly on incomplete real-world 3D scans** (ScanNet++, ARKitScenes), not on synthetic complete meshes. The trick is *visibility-guided masking*: the TSDF representation distinguishes *unobserved* voxels (sentinel value `−3 × voxel_size`) from *empty* voxels, and both the sparse VAE encoder and the flow-matching velocity loss explicitly mask out unobserved tokens so the model never learns to predict "empty" for "I never saw this region". Layout conditioning is open-vocabulary via CLIP-painted 3D maps; completion is implemented as a ControlNet branch over the frozen pretrained generator with a self-supervised frame-drop pretext task.

## Problem

Existing 3D generative models (BlockFusion, LT3SD, WorldGrow, scalable 3D DiTs) are trained on **synthetic** datasets (3D-FRONT, Objaverse) because flow-matching / diffusion losses require ground-truth complete geometry. This caps their domain coverage at curated furniture and isolated objects — the long tail of real cluttered indoor scenes (kitchens, offices, bedrooms with thousands of categories) is unreachable. Naively training on real partial scans fails: the model treats *unobserved* regions (e.g. under a table, inside a closet) as *empty* and learns the wrong distribution. Seen2Scene's contribution is the first principled fix.

## Method

Three-stage architecture:

**Stage 1 — Visibility-aware structured representation (§3.1).** Sample 256³ TSDF patches at 1.1 cm voxel size, reconstructed via VDBFusion from raw depth sequences. TSDF voxels carry three states: **surface** (|d| < 3 × voxel_size, near the zero-level set), **empty** (d ≥ truncation, free space the camera saw), **unknown** (d = sentinel `−3 × voxel_size`, never observed). The sentinel encoding is the foundational data trick — visibility is recoverable directly from the TSDF without auxiliary masks.

**Stage 2 — Masked sparse VAE (§3.2).** Sparse-conv VAE (3 stages, 64→512 channels, 8× downsampling, 8 latent channels). *Structure-aware masking*: the encoder retains only known voxels (surface or empty); unobserved voxels are replaced with a *learnable empty embedding* before encoding. KL-regularized latent. Decoder mirrors with two heads — a category head (focal-loss surface-vs-empty classifier) and a tanh TSDF regressor. The visibility mask propagates to the latent grid for downstream use.

**Stage 3 — Visibility-aware masked flow matching (§3.3).** Sparse DiT (28 transformer blocks, RoPE positional encoding) generates geometry latents conditioned on a 3D semantic layout. Layout `𝓑 = {(centroid, size, CLIP_label)}` is encoded by *painting* each box's CLIP-encoded label into a 3D layout map spatially aligned with the latent grid → tokenized → joint self-attention with geometry tokens. Loss:

$$\mathcal{L}_{\mathrm{gen}} = \mathbb{E}_{\mathbf{z}_0 \sim p_0,\, \mathbf{z}_1 \sim p_{\mathrm{geo}},\, t \sim \mathcal{U}[0,1]}\left[ \|\, \mathcal{G}_\psi(\mathbf{z}_t, t, \mathcal{B}) - (\mathbf{z}_1 - \mathbf{z}_0) \,\|^2 \right]$$

where `z_t = (1−t) z_0 + t z_1`. **Critically, geometry tokens corresponding to unobserved regions are excluded from the loss expectation**, so the model learns the velocity field only on observed regions and remains agnostic to missing areas. Classifier-free guidance via random `𝓑` dropout during training.

**Stage 4 — Scan completion via ControlNet (§3.4).** For completion, the pretrained generator `𝒢_ψ` is frozen; a ControlNet branch `𝒞_ϕ` initialized from `𝒢_ψ` injects partial-scan conditioning. *Self-supervised pretext*: from existing partial scan `v`, randomly drop a portion of depth frames to create a *more partial* `v_p`; encode `v_p` through the frozen VAE as condition; supervise with `v` as target. No complete ground-truth mesh ever required.

Inference: 50-step Euler sampling, classifier-free guidance scale 3.0. House-scale completion via overlapping 256³ patch tiling with weighted-average fusion in overlap regions.

## Results

**Scan completion (Table 1)**: vs SG-NN [Dai 2020] and NKSR [Huang 2023] on 3D-FRONT (CD), ScanNet++ + ARKitScenes (L2 / TMD / U3D-FPD). Seen2Scene with object boxes wins; Seen2Scene without boxes ties or beats baselines on most metrics. Object boxes hurt L2 slightly on observed surfaces but help recover large unobserved regions (e.g. occluded chair seats in Fig. 4).

**Layout-conditioned scene generation (Table 2)**: vs BlockFusion (layout-conditioned), LT3SD, WorldGrow (unconditional). DINOv2 FID + Uni3D FPD + Qwen3-VL VLM score. Seen2Scene full model: VLM avg **5.70**, beating LT3SD (previous best). VLM dimensions: Geometric Quality, Structural Logic (5.84), Semantic Coherence, Content Density.

**Ablations (Table 3 — masked training)**: removing visibility masking sharply degrades both reconstruction L1/L2/CD and generation U3D-FPD. Without masked training the model "wrongly treats unobserved space (under the table) as empty space and is trained to generate a wrong distribution." This is the single most load-bearing ablation in the paper.

**Ablations (Table 4 — semantic encoding)**: CLIP-encoded semantic labels significantly outperform one-hot category embeddings on real datasets — necessary for cross-dataset training (3D-FRONT 99 categories vs ScanNet++ 2,877 vs ARKitScenes 2,466). LLM-generated label-synonym augmentation adds further robustness.

**Implementation**: 4× NVIDIA H100, AdamW lr 1e-4, batch 64, BF16 mixed precision. fVDB sparse-conv backbone. Cosine annealing, 1000 warmup steps.

## Why it matters

This paper is the entry point for a new thread, [[3d-scene-completion]] (created 2026-04-24): generative completion of partial real-world scans, distinct from [[generative-3d-from-2d-priors]] (which goes from 2D images to 3D) and from [[gaussian-to-mesh-pipelines]] (which extracts meshes from radiance fields). The visibility-guided masking is the first principled mechanism in the wiki for training generative 3D priors directly on the millions of partial RGB-D scans the field already has — bypassing the synthetic-data ceiling that bounds BlockFusion / LT3SD / WorldGrow / scalable 3D DiTs.

Cross-thread, the visibility-aware encoding (3-state TSDF + structure-aware masking) is a primitive that other voxel-based pipelines can adopt. Specifically, [[lifting-foundation-models-to-3d]]'s online-voxel-feature-fusion stage currently has no notion of "unobserved" — it conflates "no feature here yet" with "empty here", which fails in occluded regions. Importing Seen2Scene's 3-state encoding is a natural Pass B bet.

The ControlNet-frozen-generator pattern with frame-drop self-supervision is a transferable pretext task: any generative model with a partial-data pretraining loss can absorb a downstream completion fine-tune without catastrophic forgetting.

## Pipeline contribution

- [[visibility-guided-masked-flow-matching_meng2026]] · candidate thread: [[3d-scene-completion]] · op_target: `op:default` · core flow-matching loss masks unobserved tokens; learns generative scene priors from partial data. Bundle with Idea B.
- [[visibility-aware-masked-sparse-vae_meng2026]] · candidate thread: [[3d-scene-completion]] · op_target: `op:default` · 3-state voxel encoder with learnable empty-embedding for unknown regions; visibility mask propagated to latent grid. Bundle with Idea A; cross-thread candidate for [[lifting-foundation-models-to-3d]] online-voxel-feature-fusion.
- [[controlnet-frozen-flow-self-supervised-completion_meng2026]] · candidate thread: [[3d-scene-completion]] · op_target: `op:default` · ControlNet branch over frozen generator + frame-drop self-supervision; turns a partial-data generative prior into a completion model without complete GT.
- [[clip-painted-3d-layout-conditioning_meng2026]] · candidate thread: [[3d-scene-completion]] · op_target: `op:default` · open-vocab layout conditioning by painting CLIP-encoded box labels into a 3D map aligned with the latent grid; reusable in any layout-conditional 3D generator.

## Relation to prior work

- Builds on flow matching ([Lipman 2022], [Lipman 2023] — no wiki page) — same paradigm as [[two-stage-latent-flow-matching-scene_chen2025]] (SAM 3D for objects), but applied to **partial scene-scale data** with the visibility-mask trick.
- Extends [SG-NN (Dai 2020)](dai2020_sg-nn.md) — SG-NN introduced self-supervised sparse generative completion from partial scans (regression-style); Seen2Scene lifts the same self-supervision principle into the modern flow-matching/ControlNet regime.
- Contrasts with [BlockFusion, LT3SD, WorldGrow] — these synthetic-only generators win on curated furniture but fail on real cluttered scenes; the synthetic-data ceiling is the explicit gap Seen2Scene closes.
- Uses [ControlNet (Zhang & Agrawala 2023)](../methods/controlnet.md) for conditioning; uses [TSDF (Curless & Levoy 1996)](curless1996_tsdf.md) representation; uses CLIP ViT-B/32 for label encoding.

## Open questions / limitations

- **Resolution ceiling**: 1.1 cm voxel + 8× VAE downsampling → fine details (chair-leg edges, fabric folds) are lost in compression. The paper's own §0.E flags this. Mitigation: hierarchical VAE or higher-res sparse grid.
- **Indoor only**: trained on indoor furniture / room-scale data. Outdoor (cars, vegetation, building exteriors) is untested; CLIP semantic vocabulary may not transfer.
- **Static scenes**: no dynamics; the visibility mask assumes single-pose-per-voxel observation.
- **No appearance / material**: pure geometry. Authors flag relighting + material as future work.
- **Patch tiling for house-scale scenes**: independent 256³ patch completion with weighted-average overlap fusion. Could produce inconsistencies at patch boundaries on long-range structures (hallways, beams). No quantitative analysis of this in the paper.
- **No isolated ablation of ControlNet vs frame-drop**: the completion mechanism (Idea C) is demonstrated by the headline comparisons but the relative contribution of ControlNet routing vs the frame-drop self-supervision pretext is not separately ablated.
- **Layout-vs-no-layout trade-off (Table 1)**: bounding-box conditioning marginally degrades L2 on observed surfaces — the model trusts the layout and can over-write observed geometry slightly. Open question: should the ControlNet branch be weighted higher relative to the layout signal in the observed regions?

## Code & license

The official repo at [github.com/quan-meng/seen2scene](https://github.com/quan-meng/seen2scene) **has no LICENSE file as of 2026-04-24** (`license_code: unknown`). Per CLAUDE.md §6.15, this does not block bet adoption or SOTA reasoning, but downstream commercial use should treat this as default-restrictive (no permission to copy / modify / distribute) until the authors publish a license. The arXiv paper itself is under the standard arXiv non-exclusive license (`license_paper: arxiv-nonexclusive`). Tracked in [License Audit](../meta/license-audit.md). `lint find-code` should re-check after 6 months.

Training data licenses are mixed: 3D-FRONT (research-only), ScanNet++ (research-only with terms of use), ARKitScenes (CC-BY-NC). Any reproduction inherits these constraints regardless of the code license.

## References added to the wiki

Created (2026-04-24):
- `wiki/papers/meng2026_seen2scene.md` (this page)
- `wiki/ideas/visibility-guided-masked-flow-matching_meng2026.md`
- `wiki/ideas/visibility-aware-masked-sparse-vae_meng2026.md`
- `wiki/ideas/controlnet-frozen-flow-self-supervised-completion_meng2026.md`
- `wiki/ideas/clip-painted-3d-layout-conditioning_meng2026.md`
- `wiki/stages/scene-completion.partial-scan-latent-encoding.md`
- `wiki/stages/scene-completion.generative-prior.md`
- `wiki/stages/scene-completion.condition-injection.md`
- `wiki/threads/3d-scene-completion.md`
- `wiki/papers/dai2020_sg-nn.md` (stub — load-bearing baseline)
- `wiki/methods/controlnet.md` (stub — load-bearing conditioning method)

Updated:
- `wiki/concepts/tsdf.md` (added visibility-aware 3-state encoding cross-link)
- `wiki/threads/gaussian-to-mesh-pipelines.md` (Pass B bet — completion as a sparse-view 3DGS post-processor)
- `wiki/threads/lifting-foundation-models-to-3d.md` (Pass B bet — visibility-aware voxel encoding for online voxel feature fusion)
- `wiki/threads/mono-depth-estimation.md` (Pass B bet — mono-depth → partial TSDF → Seen2Scene)
- `wiki/threads/generative-3d-from-2d-priors.md` (Pass B bet — Lyra-2.0 video-world geometry → Seen2Scene completion)

Not created (deferred per §6.5):
- 3D-FRONT, ScanNet++, ARKitScenes dataset pages — single-paper references for now; promote when a second paper uses them.
- People pages for Nießner / Dai / Chen — earned but deferred to a dedicated pass.
- Wan / fVDB / VDBFusion / Qwen3-VL — implementation libraries, outside scope.
- NKSR (Huang 2023) — baseline only; promote on a second mention.
- BlockFusion / LT3SD / WorldGrow — generation baselines; promote when one is the SOTA filler of a thread node.

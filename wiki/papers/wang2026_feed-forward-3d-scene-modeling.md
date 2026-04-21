---
title: "Feed-Forward 3D Scene Modeling: A Problem-Driven Perspective"
type: paper
tags: [survey, feed-forward, 3d-reconstruction, problem-driven-taxonomy, feature-enhancement, geometry-awareness, model-efficiency, augmentation, temporal-aware, 4d-reconstruction, world-models]
created: 2026-04-21
updated: 2026-04-21
sources: []
local_paper: papers/fundamentals/wang_2026_feed-forward-3d-scene-modeling.pdf
url: https://arxiv.org/abs/2604.14025
code: https://github.com/ziplab/Awesome-Feed-Forward-3D
license_paper: CC-BY-NC-SA-4.0
license_code: unknown
status: draft
---

📄 [Full paper](../../papers/fundamentals/wang_2026_feed-forward-3d-scene-modeling.pdf) · [arXiv](https://arxiv.org/abs/2604.14025) · [awesome-list repo](https://github.com/ziplab/Awesome-Feed-Forward-3D) · [project page](https://ff3d-survey.github.io)

_Paper license: `CC-BY-NC-SA-4.0` (non-commercial, share-alike) · Repo: awesome-list, no code license declared_

## TL;DR

A 66-page second-wave survey of feed-forward 3D scene reconstruction (models that map images → 3D in one forward pass). Its distinguishing move — and its only real claim to originality versus the earlier [Zhang et al. 2025](zhang2025_feed-forward-3d-survey.md) survey — is the taxonomy: Wang 2026 organizes methods by **the engineering problem they solve** (feature enhancement, geometry awareness, model efficiency, augmentation, temporal-awareness), explicitly dropping the output-representation axis (NeRF / 3DGS / Pointmap / mesh / SDF). The orthogonality claim — "methods built upon NeRF, 3DGS, or Pointmap may appear in any of the five directions" — makes the problem axis the dominant design lens. The survey also contributes a future-directions checklist (§7) that introduces two framings new to the wiki: *video-world vs 3D-world model paradigms* and *the reconstruction-vs-generation spectrum*.

## Problem

Prior compilations of feed-forward 3D (awesome-lists, the Zhang 2025 survey) group by *output representation*. This silos methods that attack the same challenge (sparse views, pose-free input, long-sequence scaling, dynamic scenes) behind unrelated output choices. Practitioners looking for "how do I handle sparse views?" have to cross-reference the NeRF, 3DGS, and pointmap sections separately. The survey argues this organization obscures the shared design motifs that actually drive progress.

## Method

The survey's structure:

1. **§2 Problem Formulation** — unifies feed-forward methods as `f(ℐ, 𝒫) → 𝒢` (images + optional poses → 3D), trained with a loss mix of geometric + photometric + regularization, inferred in a single forward pass.
2. **§3 Representations** — brief review of NeRF, 3DGS, Pointmap, and others (mesh, SDF, occupancy, voxel). Explicitly sets up the move to abstract over this axis.
3. **§4 Research Directions** — the five-axis problem-driven taxonomy (the central contribution):

| Axis | Sub-axes | Representative methods |
|---|---|---|
| 4.1 Feature Enhancement | Architectures · Cross-View Fusion · Integration of Visual Foundation Models | PixelNeRF, AttnRend, eFreeSplat, LGM, iLRM; [[CroCo]]/[[dinov2\|DINOv2]]/CLIP/UniMatch backbones |
| 4.2 Geometry Awareness | Explicit Geometric Aggregation · Post Refinement · Pose-Free Reconstruction | MVSNeRF (cost volumes), epipolar attention; Splatt3R, NoPoSplat, PF3plat, FreeSplatter, FLARE, RegGS |
| 4.3 Model Efficiency | Feature Efficiency · Representation Compaction | ENeRF, ProNeRF, Long-LRM, iLRM, ZPressor, TinySplat, FastVGGT, QuantVGGT, SparseVGGT; GGN, PixelGaussian, FreeSplat++, LongSplat |
| 4.4 Augmentation Strategies | Data Augmentation · Visual Augmentation | Puzzles, MegaSynth, Aug3D; diffusion-based NVS polish |
| 4.5 Temporal-aware Models | Online Streaming · Offline Processing · Interactive | Cut3R, Stream3R, LongStream, StreamSplat, DGS-LRM; L4GM, 4D-LRM, MonST3R, Easi3R, 4DGT; PIXIE, PhysGM |

4. **§5 Datasets & Benchmarks** — enumerates geometry-oriented (ScanNet, ETH3D) and visual-oriented (RealEstate10K, DL3DV, NeRF-Synthetic) splits; flags that most lack 3D ground truth.
5. **§6 Applications** — autonomous driving, robotics, scene understanding, SfM/SLAM, video generation, visual localization. Highlights convergence of SfM and SLAM into unified feed-forward pipelines (VGGSfM, Light3R-SfM, MASt3R-SLAM, SLAM3R, VGGT-SLAM, ARTDECO, MASt3R-Fusion, ViSTA-SLAM).
6. **§7 Future Directions** — the second valuable chunk of the paper. Six threads: rigorous benchmarks, system efficiency, scalable representations, world models, unified perception and reconstruction, open questions (reconstruction-vs-generation spectrum; feed-forward vs. lightweight per-scene tuning balance).

The **orthogonality claim** — that methods targeting the same problem may adopt any representation, and vice versa — is the testable meta-claim that justifies the re-taxonomization.

## Results

Survey papers do not report experimental results. The paper presents:

- A taxonomic figure (Fig. 2) enumerating representative methods under each of the 5 axes and sub-axes.
- An encoder-taxonomy figure (Fig. 3) showing how recent methods mix ViT/ResNet/U-Net/Mamba backbones with large-scale pre-trained priors.
- Geometry-aware pipeline figure (Fig. 4), augmentation comparison (Fig. 6), and temporal-aware categorization (Fig. 7).

## Why it matters

The wiki already covers this field's literature via [Zhang et al. 2025](zhang2025_feed-forward-3d-survey.md) and individual paper pages. Wang 2026's value is **orthogonal**:

- The **problem-axis** is structurally closer to the wiki's stage-filler idea schema than Zhang 2025's representation-axis. When an idea page declares `stages: [feed-forward-3d.pose-free-reconstruction]`, that's more naturally indexed by Wang 2026's §4.2.3 than by any of Zhang 2025's five representation buckets.
- §7.4's **video-world vs 3D-world model** distinction and §7.6's **reconstruction-vs-generation spectrum** are framings the wiki had not articulated from prior sources alone.
- §4.3's efficiency sub-taxonomy (Feature Efficiency vs Representation Compaction) names the exact design dimensions the [[feed-forward-structure-from-motion]] thread's TTT work (LoGeR, ZipMap, TTT3R) occupies — and surfaces a parallel VGGT-efficiency family (FastVGGT/QuantVGGT/SparseVGGT) not yet tracked.

## Pipeline contribution

Survey papers contribute **structural scaffolding, not pipeline components**. Specific contributions of this paper to the wiki:

- **Problem-driven 5-axis taxonomy**, captured as a first-class concept page [[feed-forward-problem-axes]]. Enables cross-thread queries like "which ideas target pose-free reconstruction?" regardless of output representation.
- **Orthogonality claim** (representation ⊥ problem-axis) — the meta-hypothesis recorded in [[feed-forward-problem-axes]]; testable once the wiki's idea pages acquire problem-axis tags.
- **Framings new to the wiki**, recorded as capability gaps / open questions on the adopting threads:
  - *Benchmark view-selection bias* (§7.1): most benchmarks use fixed view splits (RealEstate10K, ACID) that allow models to over-fit to specific viewpoint patterns; systematic viewpoint-gap stratification is missing. → new capability gap in [[feed-forward-structure-from-motion]].
  - *Reconstruction-vs-generation spectrum* (§7.6): where to observe vs. hallucinate under occlusion / sparse sampling — a continuum, not a dichotomy. → reinforces the existing "reconstruction vs generation paradigm" capability gap in [[generative-3d-from-2d-priors]] and raises it as an open question in [[feed-forward-structure-from-motion]].
  - *Video-world vs 3D-world model paradigm split* (§7.4): video diffusion models as implicit world simulators vs. persistent 3D state as the canonical world representation — feed-forward 3D positioned as the backbone for the latter. → new open question in [[generative-3d-from-2d-priors]] and [[radiance-field-evolution]].
- **Search targets for future ingest** (methods mentioned once or twice in the survey but whose mechanism sounds load-bearing and is not yet in the wiki):
  - `LongStream` — gauge-decoupled streaming visual geometry with keyframe-relative poses and cache-consistent training for very long sequences. Directly relevant to the long-context sequence-level TTT frontier in [[feed-forward-structure-from-motion]].
  - `Stream3R` — decoder-only causal Transformer for pointmap prediction; an architectural variant distinct from [[vggt|VGGT]] (full attention) and [[CUT3R]] (recurrent) within the unified Tokenize→Update→Read→De-tokenize framing.
  - `FastVGGT` / `QuantVGGT` / `SparseVGGT` — VGGT-efficiency family (training-free token merging / post-training quantization / block-sparse attention). A parallel direction to the TTT scaling story for long-context feed-forward pointmap prediction.
  - `MegaSynth` — procedural non-semantic scene generation; a data-augmentation principle worth its own page if a second paper validates "low-level geometric diversity alone suffices."
  - `4DGT` / `L4GM` / `4D-LRM` / `StreamSplat` / `DGS-LRM` — 4D Gaussian feed-forward models; a cluster worth tracking as a [[radiance-field-evolution]] extension if the wiki ever grows a 4D operating point.
- **No component-level (idea-page) pipeline contribution.** The survey is thread-level evidence; mechanism-level novelty lives on the cited papers, not on this one.

## Relation to prior work

- **Extends** [Zhang et al. 2025](zhang2025_feed-forward-3d-survey.md) — same field, orthogonal taxonomy axis. Both survive; neither supersedes.
- **Foundation methods referenced** that the wiki already covers: NeRF, [[3d-gaussian-splatting]], [[dust3r|DUSt3R]], [[mast3r|MASt3R]], [[vggt|VGGT]], [[CUT3R]], PixelNeRF, PixelSplat, MVSplat, LRM, [[CroCo]], [[dinov2|DINOv2]], MonST3R, [[zhang2025_loger|LoGeR]], [[jang2025_pow3r|Pow3R]], [[wang2025_moge|MoGe]].

## Open questions / limitations

- **The orthogonality claim is asserted, not tested.** Wang 2026 does not quantify cross-representation overlap (e.g., methods per cell in a 5×5 problem × representation matrix). The claim is the taxonomy's central thesis but is argued only qualitatively.
- **Coverage skew.** The survey prioritizes NeRF / 3DGS / Pointmap; traditional voxel/mesh/occupancy/SDF feed-forward methods get a single disclaiming paragraph in §4 but no sub-taxonomy.
- **Mechanism depth is uneven.** §4.2.1 (cost volumes) and §4.5 (temporal) treat specific mechanisms carefully; §4.4 (visual augmentation, generative priors) and §7 (future directions) are more surface-level — they identify important directions without pinning down specific mechanisms.
- **No ablations isolate the taxonomy.** Unlike a contribution paper, there are no experimental knobs — the claim stands or falls on whether the axes feel more useful than the representation axis. For the wiki, this is fine; the concept page [[feed-forward-problem-axes]] will be proven or disproven by whether it makes idea-indexing easier in practice.
- **Benchmark-rigor section (§7.1) lacks a concrete proposal.** The critique of view-selection bias is valid but the remedy ("standardized protocols, varying difficulty") is gestural.

## Code & license

- **Paper license**: `CC-BY-NC-SA-4.0` — **non-commercial**. More restrictive than the standard arXiv license of Zhang 2025 (`CC-BY-4.0`). Commercial reuse of the taxonomy text / figures would require author consent; citation and discussion is unaffected.
- **Repo**: `github.com/ziplab/Awesome-Feed-Forward-3D` — curated awesome-list of cited methods, not a runnable codebase. No declared license on the repo (as of 2026-04-21); treat `license_code` as `unknown`. No downstream use is blocked by this; the repo is a reading pointer.

## References added to the wiki

- [feed-forward-problem-axes](../concepts/feed-forward-problem-axes.md) — new concept page capturing Wang 2026's 5-axis taxonomy as cross-cutting vocabulary.
- Thread updates: [feed-forward-structure-from-motion](../threads/feed-forward-structure-from-motion.md), [radiance-field-evolution](../threads/radiance-field-evolution.md), [generative-3d-from-2d-priors](../threads/generative-3d-from-2d-priors.md) — added to `sources:`, evidence / capability gaps / open questions refreshed where applicable.

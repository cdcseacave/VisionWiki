---
title: "Lyra 2.0: Explorable Generative 3D Worlds"
type: paper
tags: [generative-3d, video-diffusion, video-world, long-horizon, 3dgs, feed-forward-reconstruction, dit, wan-2.1, spatial-memory, self-augmentation, dmd, nvidia-toronto]
created: 2026-04-21
updated: 2026-04-21
sources: []
local_paper: papers/radiance-fields/shen_2026_lyra-2-explorable-3d-worlds.pdf
url: https://arxiv.org/abs/2604.13036
code: https://github.com/nv-tlabs/lyra
license_paper: arxiv-nonexclusive
license_code: Apache-2.0
status: draft
---

📄 [Full paper](../../papers/radiance-fields/shen_2026_lyra-2-explorable-3d-worlds.pdf) · [arXiv](https://arxiv.org/abs/2604.13036) · [project page](https://research.nvidia.com/labs/sil/projects/lyra2/) · [code](https://github.com/nv-tlabs/lyra) · [HF weights](https://huggingface.co/nvidia/Lyra-2.0)

_Paper license: `arxiv-nonexclusive` · Code license: `Apache-2.0`_

## TL;DR

Lyra 2.0 is a **video-world generative reconstruction** framework: from a single input image and a user-defined long-horizon camera trajectory, it autoregressively generates a 3D-consistent walkthrough video and lifts each chunk to a 3D Gaussian Splatting representation + surface mesh ready for real-time rendering and embodied-AI simulation (Isaac Sim). Two named failure modes of long-horizon autoregressive video generation — **spatial forgetting** and **temporal drifting** — are addressed by orthogonal mechanisms: (i) a per-frame 3D cache + visibility-score retrieval + appearance-neutral canonical-coordinate correspondence injection (anti-forgetting), and (ii) a lightweight self-augmentation training strategy that exposes the model to its own degraded outputs (anti-drifting). Feed-forward reconstruction uses a modified Depth Anything v3 with a 4×-downsampled Gaussian head fine-tuned on generated scenes, followed by hierarchical sparse-grid mesh extraction. DMD distillation retaining self-augmentation gives a ~13× faster student.

## Problem

**Generative reconstruction** — generate a camera-controlled video walkthrough + lift to 3D via feed-forward reconstruction — is the established route (Lyra 1, Gen3C) to creating explorable 3D environments from minimal input. Scaling it to **large, complex environments** — multi-room apartments, long city streets with revisits — requires 3D-consistent video generation over long trajectories with substantial viewpoint changes. Current video models fail in this regime with two specific failure modes:

1. **Spatial forgetting**. As the camera explores, previously observed regions inevitably exceed the model's finite temporal context window. Revisiting those areas forces the model to hallucinate structures from scratch — breaking global layout consistency.

2. **Temporal drifting** (observation bias). Autoregressive generation accumulates per-step synthesis errors (color shifts, blurring, distortions). These compound over autoregressive steps, gradually distorting scene appearance and geometry. Drift is exacerbated during exploration because continuously introduced unseen regions dilute visual overlap with early history frames.

**Prior mitigations fall short**:
- *Global 3D memory* (Gen3C [81], SPMem [117], Lyra 1 [2]) accumulates history into a single scene representation and conditions on its rendered views. Tightly coupled — generative artifacts degrade 3D geometry, which corrupts future conditioning (error amplification).
- *Pose-embedded history frames in context* (ray-based conditioning [26]) avoids corrupted 3D intermediaries but relies entirely on self-attention to infer long-range geometric correspondence, which fails under large viewpoint change.
- *Extended temporal context* (FramePack [135]) alleviates drift by anchoring on the initial image but does not close the train-test observation-bias gap.

Lyra 2.0's design choice is to **decouple geometric tracking from pixel synthesis**: use an explicit 3D proxy *only* for information routing (which frames, what correspondence); let the diffusion prior handle all appearance synthesis. This is the `[[information-routing-vs-3d-rendering-memory]]` argument.

## Method

**Backbone**: Wan 2.1-14B DiT [107] at 832×480, causal VAE (8× spatial / 4× temporal downsampling), rectified flow matching. FlowUniPC multistep scheduler (35 steps) at inference; classifier-free guidance scale 5.0.

### 4.2 Anti-forgetting: per-frame 3D cache + geometry-aware retrieval + canonical-coordinate correspondence

**3D cache** `C` stores per frame: full-resolution depth `D_i` + camera `(T_i, K_i)` + a downsampled point cloud `P_i` at subsample `d=8` for retrieval. Per-frame independent — no global fusion.

**Visibility-score retrieval**: project every `P_i` onto the target camera; min-depth per-pixel for occlusion; visibility `φ(i) = |points within δ=0.1 of min|`; inference greedy coverage picks `N_s = 5` frames; training samples proportional to `φ(i)` for robustness.

**Context layout** for each autoregressive step:
$$\underbrace{f_1 k_1}_{\text{anchor}} \ \ \underbrace{f_4 k_2 \ f_1 k_1}_{\text{spatial slots}} \ \ \underbrace{f_{16} k_4 \ f_2 k_2 \ f_1 k_1}_{\text{temporal slots (FramePack)}} \ \ \underbrace{g_{20}}_{\text{generate}}$$

**Canonical-coordinate warping** (the communication mechanism): for retrieved frame `j`, build `C_j ∈ [-1, 1]^{3×H×W}` with channels `(u, v, 2j/N_s − 1)`; forward-warp via full-res depth:
$$\hat{C}_j = \text{FwdWarp}(C_j, D_j^s, T_j^s, T^*, K_j^s, K^*)$$
stack warped depth as 4th channel. Encode via pixel-shuffle + sinusoidal positional encoding + learned MLP aggregator. **Inject on Q and K only** (not V) in every DiT self-attention block: `q = W_Q(x + p)`, `k = W_K(x + p)`, `v = W_V(x)`. Aggregator zero-initialized.

Why *canonical coords* and not warped RGB: warped RGB has disocclusion holes, stretching, depth-boundary bleeding — the model latches onto those artifacts (warped image as "a crutch that bypasses the generative prior"). Canonical coords carry only the geometric correspondence without appearance content.

See [[per-frame-3d-cache-retrieval_shen2026]] + [[canonical-coord-warp-injection_shen2026]] for mechanism.

### 4.3 Anti-drifting: self-augmented history training

Root cause: **observation bias** — train on clean GT history, infer on own degraded outputs. Self-Forcing [36] fixes this by running full bi-directional denoising (35 steps) per history segment inside training — prohibitive on a 14B DiT.

Lyra's **lightweight proxy**: with probability `p_aug = 0.7`, sample `t ∼ 𝒰(0, 0.5)` and corrupt the history latent via the flow-matching schedule:
$$z_t^{hist} = (1-t) \cdot z_0^{hist} + t \cdot \epsilon, \quad \epsilon \sim \mathcal{N}(0, I)$$
Train the denoising objective conditioning on the corrupted history. One-step noise is a cheap proxy for the type of degradation (color shifts, blurring) autoregressive inference produces — enough to close the observation-bias gap at negligible training cost.

See [[self-augmented-history-training_shen2026]].

### 4.4 3D Reconstruction

**Feed-forward 3DGS**: Depth Anything v3 (DAv3) with a **Gaussian DPT head modified to output a `k × k` downsampled feature map** (`k=2`, 4× fewer Gaussians for streaming), fine-tuned on 3,000 autoregressive Lyra-generated one-minute scenes (10K iterations, LR 5×10⁻⁵, batch 8). The fine-tune recipe inherits from [[bahmani2025_lyra|Lyra 1]]. See [[downsampled-gaussian-dpt-head_shen2026]].

**Mesh extraction**: hierarchical sparse grid (OpenVDB [72, 115]) with adaptive level/voxel-size allocation; SDF from 3DGS depth; marching cubes per level; merge at level transitions. See [[hierarchical-sparse-grid-mesh-extraction_shen2026]].

### 4.5 Distilled student

DMD distillation [126]: 35 steps → 4 steps + CFG-distilled into single-pass. **Self-augmentation retained during distillation** so the student inherits drift robustness. ~13× per-step speedup (194s → 15s per 80-frame autoregressive step on GB200). See [[dmd-with-self-aug_shen2026]].

### Training details (App. A)

- Dataset: **DL3DV** [60] (10K real-world video clips). Camera poses via ViPE [35]; per-frame depth via DAv3 [58]; captions via Qwen3-VL-8B-Instruct [103].
- Mix: 30% image-to-video (first L=80 frames from single image); 70% autoregressive chunk (sample segment s, history [0, s·L+1), predict next L frames).
- Hardware: **64 NVIDIA GB200 GPUs**, batch 64, AdamW LR 3×10⁻⁵, wd 0.1, bf16 mixed precision, 7,000 iterations. Zero-init of all newly added modules.

## Results

### Long video generation (Table 1, single-image → ~800-frame video)

On **Tanks-and-Temples** (out-of-domain), Camera Controllability / Style Consistency / FID:
| Method | Camera Ctrl.↑ | Style Consist.↑ | FID↓ |
|---|---|---|---|
| GEN3C [81] | 70.91 | 75.54 | 79.07 |
| CaM [128] | 31.86 | 82.83 | 59.20 |
| SPMem [117] | 45.07 | 79.68 | 60.11 |
| VMem [51] | 0.00 | 70.54 | 136.48 |
| HY-WorldPlay [37] | — | 48.22 | 163.54 |
| **Ours** | **63.87** | **85.07** | **51.33** |
| Ours DMD (4-step) | 58.12 | 78.91 | 49.71 |

Lyra 2.0 wins or ties on nearly every metric on both DL3DV (in-domain) and Tanks-and-Temples. GEN3C wins on Camera Controllability alone (70.91) by virtue of rigid depth-warped conditioning — which degrades generation quality visibly (SC 75.54, well below Lyra).

### 3D scene reconstruction (Table 2)

All baselines + DAv3 vs Ours + DAv3 vs Ours Full (fine-tuned DAv3) on Tanks-and-Temples:
| Method | LPIPS-G↓ | FID↓ | Subj.Qual.↑ |
|---|---|---|---|
| SPMem + DAv3 | 0.666 | 94.11 | 9.95 |
| CaM + DAv3 | 0.693 | 94.02 | 9.79 |
| Ours + DAv3 | 0.648 | 79.36 | 14.42 |
| **Ours Full** | **0.629** | **72.47** | **18.80** |

The "Ours Full" gap over "Ours + DAv3" validates the fine-tune-on-generated-data recipe (Lyra 1 precedent, not novel here).

### Ablation (Table 3, Tanks-and-Temples — the load-bearing evidence table)

| Variant | CC↑ | Style Consist.↑ | Reproj Err↓ |
|---|---|---|---|
| Ours (full) | **63.87** | **85.07** | **0.069** |
| w/ Global Point Cloud (replaces both retrieval + canonical-coord) | 49.86 | 82.42 | 0.067 |
| w/ Explicit Corr. Fusion (replaces learned MLP aggregation) | 57.29 | 83.28 | 0.071 |
| w/o FramePack (removes temporal slots) | 62.62 | 80.61 | 0.079 |
| w/o Self-Augmentation | 53.92 | 77.98 | 0.066 |

The "w/ Global Point Cloud" row is the **−14.01 CC** swing that isolates Ideas 1+2 as a bundle against the global-3D-memory alternative. "w/o Self-Augmentation" isolates Idea 3 — and also exposes the honest trade-off: per-frame Subjective Quality actually *improves* 43.35 → 47.88 without augmentation, at the cost of long-horizon consistency.

### Distilled student (Table 1 "Ours DMD" row)

LPIPS/FID parity or slight improvement vs the 35-step teacher; Camera Controllability degrades moderately (63.87 → 58.12). Still SOTA above all non-Lyra baselines, ~13× faster per step.

## Why it matters

Lyra 2.0 is the **canonical in-wiki representative of the video-world paradigm** for generative 3D scene creation. The [[generative-3d-from-2d-priors]] thread previously tracked only 3D-world methods (SAM 3D, latent flow-matching); Wang 2026 flagged the missing video-world paradigm as a capability gap. This paper fills that gap with a well-isolated mechanism-level contribution:

1. **Consolidates three separate subsystems** (long-horizon memory, drift mitigation, fast inference) under the single design principle that the 3D proxy is for routing, not rendering. The principle gets its own concept page: [[information-routing-vs-3d-rendering-memory]].
2. **Ideas with cross-thread transferability**:
   - `[[self-augmented-history-training_shen2026]]` is architecture-agnostic and applies to any autoregressive flow/diffusion model — likely to recur in Pass B bets across threads.
   - `[[canonical-coord-warp-injection_shen2026]]` encodes the "Q/K-only zero-init injection onto a pretrained backbone" pattern — a general recipe for adding guidance to large pretrained models without destabilizing them.
3. **Enables downstream applications the wiki's other threads care about**: the mesh output plugs into [[gaussian-to-mesh-pipelines]] at a scale none of its existing fillers have demonstrated; the 3DGS output fits [[radiance-field-evolution]]'s `op:city-scale` as a candidate for streaming-compatible representation.
4. **Exposes a weakness in ablation coverage**: Idea 2 (canonical-coord vs warped-RGB) and Ideas 4 + 6 lack isolated ablations. Flagged on the idea pages.

## Pipeline contribution

Six first-class ideas extracted in Step 3 of this ingest:

- [[per-frame-3d-cache-retrieval_shen2026]] · candidate thread: [[generative-3d-from-2d-priors]] · op_target: new `op:explorable-scene` · mechanism: per-frame 3D cache + visibility-score greedy-coverage retrieval; `N_s=5` · expected gain: recovers revisit consistency beyond temporal window (bundle with Idea 2: +14.01 Camera Controllability on T&T vs global-point-cloud baseline).
- [[canonical-coord-warp-injection_shen2026]] · candidate thread: [[generative-3d-from-2d-priors]] · op_target: `op:explorable-scene` · mechanism: forward-warp appearance-free canonical coordinate maps via full-res depth; inject on Q/K only of DiT self-attention; zero-init aggregator · expected gain: dense geometric alignment without RGB-artifact propagation (bundled with Idea 1).
- [[self-augmented-history-training_shen2026]] · candidate thread: [[generative-3d-from-2d-priors]] · op_target: `op:explorable-scene` · mechanism: probability-`p_aug=0.7` one-step noise injection on clean-latent history via flow-matching schedule · expected gain: +7–10 pts on long-horizon Style Consistency + Camera Controllability (isolated ablation); ~35× cheaper than Self-Forcing on bi-directional DiTs. **Cross-thread transfer candidate.**
- [[downsampled-gaussian-dpt-head_shen2026]] · candidate thread: [[radiance-field-evolution]] `op:city-scale` or [[generative-3d-from-2d-priors]] `op:explorable-scene` · mechanism: `k × k` strided output on per-pixel Gaussian DPT head (k=2, 4× Gaussian reduction) · expected gain: streaming-compatible Gaussian count at real-time rendering budget. **Evidence weaker — no isolated ablation.**
- [[dmd-with-self-aug_shen2026]] · candidate thread: [[generative-3d-from-2d-priors]] `op:explorable-scene` · mechanism: DMD 35→4 step distillation + CFG distilled + self-aug retained during distillation · expected gain: ~13× per-step speedup; student remains above all non-Lyra baselines.
- [[hierarchical-sparse-grid-mesh-extraction_shen2026]] · candidate thread: [[gaussian-to-mesh-pipelines]] · op_target: candidate for a large-scale OP (currently none; would motivate a new one if evidence compares favorably vs MILo/CoMe at scale) · mechanism: adaptive-level OpenVDB grid + per-level marching cubes + transition merging · expected gain: scene-scale mesh extraction; isolated quantitative comparison absent in this paper.

Mechanism-level details live on the idea pages; this section is cross-reference only.

## Relation to prior work

- **Direct lineage**: [[bahmani2025_lyra|Lyra 1]] (Bahmani et al., Sep 2025) — establishes the generative-reconstruction paradigm + the fine-tune-on-generated-data recipe. Lyra 2.0 inherits both, adds the long-horizon mechanisms.
- **Global-3D-memory family** (contrasted against): Gen3C [81], SPMem [117], Lyra 1 [2], WorldMem [118]. The [[information-routing-vs-3d-rendering-memory]] concept page makes the distinction explicit.
- **Retrieval-based memory family** (refines): Context-as-Memory [128], VMem [51], WorldMem [118]. Lyra adds geometry-aware retrieval (visibility score) + dense correspondence injection (canonical coords) — these prior methods lack the latter.
- **Drift mitigation**: refines Self-Forcing [36] (causal-only) into a bi-directional-applicable, compute-cheap variant.
- **Video-diffusion backbone**: Wan 2.1-14B [107].
- **Feed-forward 3DGS**: Depth Anything v3 [58] — see [[depthanythingv3]] stub.
- **Context compression**: FramePack [135] — adopted as-is.
- **Pose conditioning**: Plücker rays [26] — adopted as-is.
- **Distillation**: DMD [126] — adopted with self-aug retention.
- **Reconstruction-vs-generation spectrum**: [[wang2026_feed-forward-3d-scene-modeling]] §7.6 — Lyra 2.0 is a canonical reconstruction-first-generation-second (Visual Augmentation mirror per §4.4.2) example.

## Open questions / limitations

**Author-stated (§6 Discussion)**:
1. **Static scenes only** — dynamic objects are not modeled. Future work.
2. **Photometric inheritance from training data** — DL3DV has exposure variation across views; the model may reproduce such inconsistency, causing artifacts in feed-forward 3DGS reconstruction. Remedies suggested: in-network photometric stabilization [16], or training on photometrically consistent synthetic data [128] (game-engine captures).

**Analyst skepticism**:
- **Ablation coverage gap**: Idea 2 (canonical-coord-vs-warped-RGB) is not cleanly isolated — "w/ Explicit Corr. Fusion" ablates the aggregation mechanism but not the appearance-neutrality choice. Ideas 4 (downsampled head) and 6 (hierarchical mesh extraction) have no isolated ablations. Flagged on their idea pages.
- **Self-augmentation distribution match**: the paper argues one-step flow-matching noise is a plausible proxy for inference-time drift but does not quantify distributional distance. If real drift is structured (spatially correlated color shifts from accumulated lighting errors) and one-step noise is i.i.d. Gaussian, the proxy may miss failure modes.
- **FramePack dependency**: "w/o FramePack" drops SC by 4.46 pts even with retrieval + self-aug active. The paper's claim that retrieval + self-aug are the *core* anti-drifting mechanism is empirically supported, but FramePack still contributes materially — the three mechanisms together, not any one alone, achieve the final numbers.
- **DL3DV-only training**: domain shift to phone captures, drone footage, or aerial imagery is untested. Qualitative in-the-wild results (Fig. 8) are promising but unquantified.
- **Per-frame 3D cache scaling**: the cache grows linearly with generated frames; no pruning described. Long sessions (many thousands of frames) may hit memory limits; eviction strategy unspecified.
- **"Retention of self-aug through distillation" claim**: Section 4.5 asserts this matters but provides no distilled-without-aug baseline. Theoretical argument only.

## Code & license

- **Paper**: `arxiv-nonexclusive` — standard, no downstream restriction for reference.
- **Code**: [github.com/nv-tlabs/lyra](https://github.com/nv-tlabs/lyra) — **Apache-2.0** (commercial-friendly). Unusual for a Wan-2.1-based system; the upstream Wan 2.1 weights likely carry their own restrictions that compose into effective deployment terms, but the Lyra code itself imposes no additional constraint.
- **Weights**: [huggingface.co/nvidia/Lyra-2.0](https://huggingface.co/nvidia/Lyra-2.0) — license not independently audited at ingest time; `lint licenses` should verify on next run.

Per §6.15, license info is informational. Bet composition and SOTA choices are unaffected.

## References added to the wiki

- [[per-frame-3d-cache-retrieval_shen2026]]
- [[canonical-coord-warp-injection_shen2026]]
- [[self-augmented-history-training_shen2026]]
- [[downsampled-gaussian-dpt-head_shen2026]]
- [[dmd-with-self-aug_shen2026]]
- [[hierarchical-sparse-grid-mesh-extraction_shen2026]]
- [[video-world.spatial-memory]] (new stage)
- [[video-world.correspondence-injection]] (new stage)
- [[video-world.drift-mitigation-training]] (new stage)
- [[video-world.distillation]] (new stage)
- [[information-routing-vs-3d-rendering-memory]] (new concept)
- [[bahmani2025_lyra]] (new paper stub — precedent)
- [[depthanythingv3]] (new method stub — load-bearing backbone)

Updated (Step 5 cascade):
- [[generative-3d-from-2d-priors]] (primary thread: new OP `op:explorable-scene`, SOTA nodes, lineage, bets, capability-gap updates)
- [[gaussian-to-mesh-pipelines]] (candidate components: hierarchical-sparse-grid extraction at scale; open question on large-scale mesh-extraction comparison)
- [[radiance-field-evolution]] (candidate components: downsampled Gaussian DPT head; open question refresh on video-world paradigm)
- [[feed-forward-structure-from-motion]] (capability-gap update: reconstruction-vs-generation spectrum now has its canonical video-world exemplar)

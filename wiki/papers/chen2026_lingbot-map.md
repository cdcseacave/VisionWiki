---
title: "LingBot-Map: Geometric Context Transformer for Streaming 3D Reconstruction"
type: paper
tags: [sfm, streaming, feed-forward, attention, slam, foundation-model, long-sequence]
created: 2026-04-24
updated: 2026-04-24
sources: []
local_paper: papers/sfm-slam/chen_2026_lingbot-map.pdf
url: https://arxiv.org/abs/2604.14141
code: https://github.com/robbyant/lingbot-map
license_paper: arxiv-nonexclusive
license_code: unknown
status: draft
---

📄 [Full paper](../../papers/sfm-slam/chen_2026_lingbot-map.pdf) · [arXiv](https://arxiv.org/abs/2604.14141) · [code](https://github.com/robbyant/lingbot-map) · [project page](https://technology.robbyant.com/lingbot-map)

_Paper license: `arxiv-nonexclusive` · Code license: `unknown` (no LICENSE file in repo as of 2026-04-24)_

## TL;DR

LingBot-Map is a feed-forward streaming 3D foundation model that predicts per-frame camera pose + dense depth online, with no access to future frames. Its central contribution is **Geometric Context Attention (GCA)**, a structured streaming attention mechanism that partitions the state into three typed contexts — an *anchor context* (first $n$ frames, fixed references for scale + coordinate grounding), a *pose-reference window* (last $k$ frames with full image tokens for local geometry), and a *trajectory memory* (6 compact tokens per evicted frame for long-range drift correction). Every component ablates as strictly positive. At inference, paged KV-cache + FlashInfer yield 20 FPS at 10K+ frames on 518×378 inputs. LingBot-Map surpasses every streaming, offline, and optimization-based baseline on long-sequence Oxford-Spires (ATE 6.42 → 7.11 over a 12× sequence-length increase, vs. CUT3R's 18.16 → 32.47).

## Problem

Feed-forward 3D foundation models ([[vggt|VGGT]], [[dust3r|DUSt3R]], Pi3, DA3) achieve strong accuracy **offline** but cannot stream: they require full frame sets and scale quadratically in $T$. Existing streaming approaches each break down differently:

- **Causal attention** (Stream3R, StreamVGGT, Wint3R) grows linearly in memory — prohibitive beyond a few thousand frames.
- **RNN-style recurrence** (CUT3R) aggressively compresses into a persistent state — forgets.
- **Hybrid SLAM backends** (VGGT-SLAM, [[murai2025_mast3r-slam|MASt3R-SLAM]]) bolt feed-forward priors onto hand-crafted keyframe/BA backends — not end-to-end differentiable.
- **Test-time training** ([[zhang2025_loger|LoGeR]], [[jin2026_zipmap|ZipMap]], [[chen2026_ttt3r|TTT3R]]) replaces attention with fast-weight recurrence — scales, but sacrifices the composable structure of geometric context.

Core thesis: the streaming state should selectively retain **what matters** (not just *how much*), and the selection criterion should be grounded in classical SLAM's structural insight — distinct spatial-context types serve distinct geometric roles.

## Method

### Geometric Context Attention (GCA)

Each frame is encoded by a DINOv2 ViT backbone (patch size 14) producing $M \approx 500$ image tokens; augmented with a camera token $\mathbf{c} \in \mathbb{R}^C$, four register tokens $\mathbf{r}_j$, and a learnable anchor token $\mathbf{a}$. Tokens flow through alternating blocks of **Frame Attention** (within-frame) and **GCA** (cross-frame, structured).

The GCA attention mask partitions the keys available to the current frame $t$ into three disjoint contexts:

1. **Anchor Context $\mathcal{A}$** — the first $n$ frames ($n=3$) with full image tokens. Training-time scale normalization:
   $$s = \frac{1}{|\mathcal{X}^{\text{anchor}}|} \sum_{\mathbf{x} \in \mathcal{X}^{\text{anchor}}} \|\mathbf{x}\|_2$$
   all GT depths and translations are divided by $s$. At inference, anchor-token representations provide a fixed coordinate frame + metric scale reference; every subsequent frame attends to them.
2. **Pose-Reference Window $\mathcal{W}$** — the last $k$ frames with full image tokens ($k=16$–$64$ at inference; trained with $k$ randomly sampled in $[16, 64]$ for robustness). Dense visual overlap for frame-to-frame pose registration.
3. **Trajectory Memory $\mathcal{M}$** — frames outside $\mathcal{A} \cup \mathcal{W}$ retain only **6 tokens each** (1 camera + 1 anchor + 4 register); image tokens are evicted. Video RoPE [ref 72, not ingested] is injected for temporal ordering.

Per-frame attention cost: $(n+k) \cdot M + 6T$ — constant first term, 6-tokens-per-frame growth in the second. Causal attention accumulates $T \cdot (M+6) = MT + 6T$. Analytical reduction: ~80× at $T=10^4$, $M=500$.

### Loss function

$$\mathcal{L} = \lambda_{\text{depth}} \mathcal{L}_{\text{depth}} + \lambda_{\text{abs-pose}} \mathcal{L}_{\text{abs-pose}} + \lambda_{\text{rel-pose}} \mathcal{L}_{\text{rel-pose}}$$

- $\mathcal{L}_{\text{depth}}$ and $\mathcal{L}_{\text{abs-pose}}$ follow VGGT's uncertainty-weighted formulation. **Unlike VGGT, $\mathcal{L}_{\text{abs-pose}}$ supervises camera-to-world extrinsics** (not world-to-camera) — claim: decouples rotation-translation error coupling in long sequences. Not ablated in isolation.
- $\mathcal{L}_{\text{rel-pose}} = \frac{1}{k(k-1)} \sum_{i \neq j} (\mathcal{L}_{\text{rot}}(i,j) + \lambda_{\text{trans}} \mathcal{L}_{\text{trans}}(i,j))$ over all ordered pairs within the pose-reference window. Geodesic rotation + L1 translation. Inspired by π³ (ref [83], not yet ingested).

### Two-stage training

- **Stage 1 — base model (21.5K GPU-hrs)**: standard global (non-GCA) attention on 2–24 view samples. Training corpus: 29 datasets spanning multi-view collections + video sequences (BlendedMVS, HyperSim, MegaDepth, TartanAir v1/v2/Ground, DL3DV, MatrixCity, ScanNet/++, internal game data, et al.). Lr 2e-4, AdamW, 160K iters, bfloat16 FSDP. Co-jittering augmentation (shared photometric transform across a scene's frames, probability 0.3) encourages geometric cues over appearance shortcuts.
- **Stage 2 — streaming fine-tune (15.4K GPU-hrs)**: replace global attention with GCA. Progressive view curriculum 24→320. Window size $k$ randomly sampled 16→64 during training. Ulysses [Jacobs 2023] context-parallelism (16-way) on TorchTitan + Magi Attention for the quadratic-attention cost. Stage-2 corpus down-weights multi-view-only datasets, up-weights long-trajectory video (TartanAir, MatrixCity, Waymo, ScanNet, KITTI-360, internal game). Introduces the **foldback video sampler**: random start, random stride, direction-flip at sequence boundary — yields subsequences with varying frame rates and no forward-time bias.

### Inference

- **Paged KV-cache** (inspired by PagedAttention [Kwon 2023]) eliminates memory-reallocation overhead from the evict/append churn inherent in streaming. FlashInfer runtime: **20.29 FPS** vs. **10.5 FPS** contiguous-PyTorch baseline (≈2× speedup) at 518×378 with 64-frame window, 1000-frame sequence.
- **Optical-flow-adaptive keyframe selection**: compute flow from the model's own predicted depth+pose to the last keyframe; threshold magnitude for keyframe inclusion. Used in both inference modes.
- **Two modes**:
  - **Direct Mode** (default): causal GCA, no state reset. Stable to ~3K frames (≈10× the 320-view training length); quality degrades gradually beyond.
  - **VO Mode**: partition into overlapping windows, reset state per window, Sim(3)-align overlap regions to fuse. Trades bounded memory for window-boundary alignment drift.

## Results

### Oxford-Spires — sparse (320 frames)

| Method | Type | AUC@15↑ | ATE ↓ | RPE-trans↓ | RPE-rot↓ |
|---|---|---:|---:|---:|---:|
| DA3 | offline | 49.84 | 12.87 | 3.22 | 16.17 |
| Pi3 | offline | 38.64 | 14.03 | 2.58 | 10.33 |
| VGGT | offline | 23.84 | 24.78 | 8.87 | 22.79 |
| VIPE | optim | 45.35 | 10.52 | **0.43** | 5.98 |
| DroidSLAM | optim | 8.58 | 21.84 | 1.02 | 6.90 |
| CUT3R | online | 5.98 | 18.16 | 1.17 | 7.18 |
| TTT3R | online | 13.92 | 19.35 | 2.28 | 13.30 |
| Wint3R | online | 11.61 | 21.10 | 1.62 | 6.27 |
| **LingBot-Map** | **online** | **61.64** | **6.42** | 1.01 | 3.70 |

LingBot-Map beats every offline and optimization-based method on AUC@15 and ATE, and is **2.8× lower ATE than the best online competitor** (CUT3R). Only VIPE's RPE-trans is sharper — and VIPE is iterative bundle adjustment.

### Oxford-Spires — dense (3840 frames, 12× longer)

| Method | ATE sparse | ATE dense | ΔATE | FPS |
|---|---:|---:|---:|---:|
| CUT3R | 18.16 | 32.47 | **+14.31** | 29.21 |
| TTT3R | 19.35 | 25.05 | +5.70 | 28.97 |
| Wint3R | 21.10 | 32.90 | +11.80 | 3.88 |
| InfiniteVGGT | 30.49 | 31.75 | +1.26 | 7.78 |
| Stream3R-w | 33.03 | 33.73 | +0.70 | 13.66 |
| **LingBot-Map** | **6.42** | **7.11** | **+0.69** | **20.29** |

LingBot-Map's ΔATE is the only one close to zero while *starting* from a ~3–5× lower baseline. This scaling curve — not the absolute number — is the headline.

### ETH3D / 7-Scenes / Tanks & Temples

LingBot-Map best ATE on all three (0.22 / 0.08 / 0.20); best F1 reconstruction (98.98 / 80.39 — Tanks numbers not in excerpts). ETH3D F1 **+21.70** over runner-up Wint3R (77.28).

### Ablation (Table 6, TartanAir + TartanGround)

| Rel-Loss | Anchor Init | Ctx Tokens | Video RoPE | AUC@3↑ | ATE↓ | RPE-rot↓ |
|:---:|:---:|:---:|:---:|---:|---:|---:|
| ✓ | | | | 9.80 | 8.59 | 2.57 |
| ✓ | ✓ | | | 13.63 | 7.88 | 2.90 |
| | ✓ | ✓ | | 13.91 | 8.25 | 5.35 |
| ✓ | ✓ | ✓ | | 15.75 | 7.46 | 2.26 |
| ✓ | ✓ | ✓ | ✓ | **16.39** | **5.98** | **1.93** |

Every component contributes monotonically on AUC@3. Video RoPE alone drives the **single largest ATE improvement** (7.46 → 5.98, −1.48 m). Relative pose loss is critical for RPE-rot (dropping it → 2.4× degradation).

### Bounded window vs. full attention (Table 7)

| Window | ATE↓ | RPE-trans↓ | RPE-rot↓ | FPS | Mem (GB) |
|---|---:|---:|---:|---:|---:|
| 64 | **5.98** | **1.33** | 1.93 | 20.29 | 13.28 |
| Full | 6.60 | 1.50 | **1.71** | 11.87 | 36.06 |

**Counter-intuitive**: bounded window beats full causal attention on ATE + RPE-trans, losing only marginally on RPE-rot. Paper hypothesis: distant image tokens add noise rather than signal. Not proven.

## Why it matters

LingBot-Map introduces a **third family of Tier-3 scaling mechanisms** for the [[feed-forward-structure-from-motion]] thread:

1. **TTT family** — LoGeR, ZipMap, TTT3R: replace attention-function-class with fast-weight recurrence.
2. **VGGT-compression family** — FastVGGT, Faster-VGGT, QuantVGGT: preserve attention-function-class, compute less of it.
3. **Structured-attention decomposition (NEW)** — GCA: preserve attention mechanism but impose geometrically-grounded eviction/partitioning.

This reshapes the thread's scaling-family comparison (Bet #030 now needs a three-way head-to-head, not two-way). It also directly addresses the "gauge-decoupled streaming" capability gap flagged from Wang 2026 §4.5.1: the anchor-frame context is a *learned* gauge-decoupling mechanism (fixed coordinate frame anchored to first $n$ frames rather than continuously updating).

Contradicts an implicit assumption shared by TTT3R/ZipMap/LoGeR: that *test-time adaptation* is necessary for long-sequence streaming. GCA shows that a purely feed-forward model (no inference-time parameter updates) can outperform the TTT family on both accuracy (ATE 7.11 vs TTT3R 25.05 at 3840 frames) and efficiency (20.29 FPS at bounded memory).

## Pipeline contribution

- [[three-context-geometric-attention_chen2026]] · candidate thread: [[feed-forward-structure-from-motion]] · stages: `feed-forward-sfm.attention-mechanism`, `feed-forward-sfm.coordinate-grounding`, `feed-forward-sfm.long-context-memory` · **composite** (co_requires the three atomic sub-ideas below); rewrites the monolithic attention slot into a coordinated trio.
- [[anchor-frame-scale-grounding_chen2026]] · target stage: `feed-forward-sfm.coordinate-grounding` (new) · portable scale-normalization via first-n-frames anchor + anchor-point-cloud-mean-distance normalization.
- [[compact-trajectory-memory-tokens_chen2026]] · target stage: `feed-forward-sfm.long-context-memory` · 6-token-per-frame eviction for constant per-frame memory growth; portable (candidate replacement for CUT3R's RNN state).
- [[video-rope-on-trajectory-tokens_chen2026]] · target stage: `feed-forward-sfm.long-context-memory` (refinement) · temporal positional encoding on compressed memory; single biggest ATE mover.
- [[sliding-window-pairwise-pose-loss_chen2026]] · target stage: `feed-forward-sfm.training-recipe` · pairwise rel-pose supervision across the local window; portable training objective.
- [[camera-to-world-pose-parameterization_chen2026]] · target stage: `feed-forward-sfm.camera-parameterization` · decouple RT error coupling; **weak evidence — no isolated ablation**, flagged on idea page.
- [[paged-kv-cache-streaming-sfm_chen2026]] · target stage: `feed-forward-sfm.long-context-memory` (inference-runtime aspect) · ~2× speedup for evict/append; portable systems-level optimization.

Non-extracted contributions mentioned for completeness (no dedicated idea page — no ablation or data-loading tweak only): optical-flow-adaptive keyframe selection using predicted depth+pose; foldback video sampler (random-stride + direction-flip for long-sequence training); two-stage progressive-view training curriculum.

## Relation to prior work

- **Builds on**: [[vggt|VGGT]] (architecture, ViT+DPT head lineage, composite loss formulation); [[dust3r|DUSt3R]] paradigm; [[leroy2024_mast3r|MASt3R]] through the feed-forward lineage. Inherits relative-pose loss formulation from π³ (ref [83], not yet ingested). Uses PagedAttention [Kwon 2023 — LLM serving] for paged KV-cache; Ulysses [Jacobs 2023] context-parallelism for training.
- **Head-to-head competes with**: [[chen2026_ttt3r|TTT3R]], [[jin2026_zipmap|ZipMap]], [[zhang2025_loger|LoGeR]] (TTT family); CUT3R, StreamVGGT, Stream3R, Wint3R, InfiniteVGGT, SLAM3R, Spann3R (causal/recurrent streaming); [[murai2025_mast3r-slam|MASt3R-SLAM]], VGGT-SLAM (hybrid SLAM).
- **Contradicts**: the implicit "TTT is required for long-sequence streaming" hypothesis of TTT3R/ZipMap/LoGeR. GCA shows a feed-forward, structured-attention model can beat the TTT family on ATE at every tested sequence length, without any inference-time parameter updates.
- **Addresses directly**: the "gauge-decoupled streaming" capability gap (Wang 2026 §4.5.1) via anchor-frame context.

## Open questions / limitations

**Authors' stated limitations**:
1. No explicit loop-closure detection; revisits are handled only implicitly by trajectory-memory retention.
2. Fixed 6-token-per-frame budget in trajectory memory may lose fine-grained geometric detail over tens of thousands of frames.
3. No test-time optimization — no adaptation to pathological inference-time conditions.

**Reader skepticism**:
- **Anchor degenerate-geometry failure mode**: if the first $n=3$ frames have negligible parallax (static camera) or near-planar geometry, the anchor scale is ill-defined. Not tested in the paper.
- **Missing register-token ablation**: are the 4 register tokens geometrically load-bearing, or noise-absorbing slack? No count-variation ablation.
- **Bounded-window-beats-full-attention mechanism**: "distant image tokens add noise" is a hypothesis not proven. Could be a stage-2 training-distribution artifact ($k$ was sampled ≤64 — the model never learns to use $k>64$).
- **Camera-to-world parameterization**: motivation is plausible but **unablated**. May be a post-hoc rationalization.
- **VO-mode alignment drift**: acknowledged but not quantified — at what sequence length does alignment drift dominate trajectory-memory drift?
- **Stage-1 corpus size (21.5K GPU-hours)**: absent this massive pre-training, GCA's contribution is unclear. What fraction of performance comes from the stage-1 prior vs. the GCA structure itself?

## Code & license

- **Repo**: [github.com/robbyant/lingbot-map](https://github.com/robbyant/lingbot-map) — present, README documents installation (PyTorch 2.8.0 + CUDA 12.8 + NVIDIA Kaolin), demo scripts, checkpoints (as of 2026-04-24).
- **LICENSE**: **no LICENSE file in the repo as of 2026-04-24** (both `main` and `master` return 404 for `/LICENSE`). Legal status is therefore unsettled. Downstream redistribution or commercial use is ambiguous; the Ant Group affiliation suggests a corporate-research release consistent with non-commercial terms, but nothing is stated. Implementation-time audit should re-check via `lint find-code` or `lint licenses` after 6 months. `license_code: unknown`.
- **Paper license**: arXiv default non-exclusive distribution license; author copyright retained. Not yet venue-published as of v2 (16 Apr 2026).

## References added to the wiki

**New**:
- `wiki/ideas/three-context-geometric-attention_chen2026.md`
- `wiki/ideas/anchor-frame-scale-grounding_chen2026.md`
- `wiki/ideas/compact-trajectory-memory-tokens_chen2026.md`
- `wiki/ideas/video-rope-on-trajectory-tokens_chen2026.md`
- `wiki/ideas/sliding-window-pairwise-pose-loss_chen2026.md`
- `wiki/ideas/camera-to-world-pose-parameterization_chen2026.md`
- `wiki/ideas/paged-kv-cache-streaming-sfm_chen2026.md`
- `wiki/stages/feed-forward-sfm.coordinate-grounding.md`

**Updated**:
- `wiki/threads/feed-forward-structure-from-motion.md` (Evidence Tier 3 + Emerging patterns + Capability gaps + Pass B bets + sources)
- `index.md`
- `log.md`

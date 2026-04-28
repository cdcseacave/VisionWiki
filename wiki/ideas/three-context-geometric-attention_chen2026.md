---
title: Three-context Geometric Context Attention (GCA)
type: idea
source_paper: wiki/papers/chen2026_lingbot-map.md
also_in: []

# pipeline shape
scope: topology-rewrite
stages: [feed-forward-sfm.attention-mechanism, feed-forward-sfm.coordinate-grounding, feed-forward-sfm.long-context-memory]
rewrites:
  replaces: [feed-forward-sfm.attention-mechanism]
  introduces: [feed-forward-sfm.coordinate-grounding]

# type contracts
inputs: [per-frame-token-stream-with-anchor-camera-register-tokens]
outputs: [attended-tokens-with-bounded-per-frame-context, absolute-camera-pose-tokens, depth-feature-tokens]
assumptions: [posed-input-not-required, streaming-input, monocular-or-multi-view, static-scene, known-intrinsics]
requires_upstream_property: [vit-backbone-with-per-frame-camera-and-register-tokens]
requires_downstream_property: [camera-head-and-depth-head-consume-post-attention-tokens]
learned_params: [anchor-token, register-tokens, attention-projections-for-three-context-partitioning]
failure_modes: [degenerate-anchor-geometry-static-first-frames, window-size-extrapolation-beyond-64, very-long-sequences-beyond-10k-frames-fine-detail-loss, no-loop-closure-for-revisits]

# composition graph
requires: []
unlocks: []
co_requires: [anchor-frame-scale-grounding_chen2026, compact-trajectory-memory-tokens_chen2026, video-rope-on-trajectory-tokens_chen2026]
bridges: []
equivalent_to: []
refines: []
contradicts: []

# housekeeping
tags: [attention, streaming, sfm, feed-forward, gca, lingbot-map]
created: 2026-04-24
updated: 2026-04-24
status: unclaimed
---

## Mechanism

GCA is a structured streaming attention that decomposes the attention context available to the current frame $t$ into **three disjoint, typed context types**, each playing a distinct geometric role:

1. **Anchor context** $\mathcal{A}$: the first $n$ frames ($n=3$ in LingBot), retained with *full image tokens* for the entire stream. Role: fix the global coordinate frame + metric scale. Mechanism: see [[anchor-frame-scale-grounding_chen2026]].
2. **Pose-reference window** $\mathcal{W}$: the last $k$ frames ($k=16$–$64$), retained with *full image tokens*. Role: dense visual overlap for per-frame pose registration via learned correspondence in attention.
3. **Trajectory memory** $\mathcal{M}$: every frame outside $\mathcal{A} \cup \mathcal{W}$, retained with only *6 compact tokens* per frame (camera + anchor + 4 register). Image tokens are evicted. Role: long-range drift correction via temporal-positional-encoded compressed history. See [[compact-trajectory-memory-tokens_chen2026]] + [[video-rope-on-trajectory-tokens_chen2026]].

The attention mask is: frame $t$'s queries attend to $\mathcal{A}$-tokens ∪ $\mathcal{W}$-tokens ∪ $\mathcal{M}$-tokens. Gradients flow end-to-end; no hand-crafted SLAM priors. Per-frame attention cost is $(n+k) \cdot M + 6T$, with $M \approx 500$ image tokens/frame — constant first term, 6-tokens/frame growth in the second. At $T=10^4$, ~80× smaller than causal attention's $MT + 6T$.

Why this shape works: classical SLAM systems independently discovered that robust real-time reconstruction requires the same three context types (reference-frame-for-grounding + active-keyframes-for-local-geometry + global-map-for-drift). GCA realizes the same decomposition end-to-end differentiably, replacing hand-crafted heuristics with learned attention weights.

**Topology-rewrite vs. stage-swap rationale**: filling the `feed-forward-sfm.attention-mechanism` slot alone doesn't capture GCA's contribution. GCA simultaneously:
- *introduces* a `feed-forward-sfm.coordinate-grounding` stage filled by the anchor context,
- *fills* `feed-forward-sfm.attention-mechanism` with the partitioned-attention mask,
- *fills* `feed-forward-sfm.long-context-memory` with the 6-token-per-frame eviction + Video RoPE.

These three stages are co-designed; adopting GCA in a thread requires claiming all three slots. The three sub-ideas are independently portable (see "Portability" below).

## Why it wins

Ablation (Table 6, TartanAir + TartanGround fine-tune, 320-frame/stride-8 eval spanning ~2,400 frames):

| Rel-Loss | Anchor | Ctx Tokens | V-RoPE | AUC@3 | ATE | RPE-rot |
|:-:|:-:|:-:|:-:|-:|-:|-:|
| ✓ | | | | 9.80 | 8.59 | 2.57 |
| ✓ | ✓ | | | 13.63 | 7.88 | 2.90 |
| ✓ | ✓ | ✓ | | 15.75 | 7.46 | 2.26 |
| ✓ | ✓ | ✓ | ✓ | **16.39** | **5.98** | 1.93 |

Every sub-component contributes strictly positively on AUC@3 and ATE. Full paper ablation isolates Video RoPE as the single largest ATE mover (7.46 → 5.98).

Head-to-head long-sequence scaling (Oxford-Spires dense, 3840 frames, 12× training-length):

| Method | ATE | ΔATE-from-sparse |
|---|-:|-:|
| CUT3R | 32.47 | +14.31 |
| TTT3R | 25.05 | +5.70 |
| Wint3R | 32.90 | +11.80 |
| Stream3R-w | 33.73 | +0.70 |
| **LingBot-Map** | **7.11** | **+0.69** |

The ΔATE is LingBot-Map's thesis: it is the only streaming method whose error stays roughly flat as the sequence grows 12×. See [[chen2026_lingbot-map]] for benchmark detail.

## Preconditioning & compatibility

- **Upstream**: requires a ViT-backbone tokenizer that produces per-frame image tokens *and* per-frame camera + anchor + register tokens. Any VGGT-style or DUSt3R-style architecture satisfies this.
- **Downstream**: camera head + DPT-style depth head consume post-attention tokens. Standard.
- **Stages filled jointly**: `feed-forward-sfm.attention-mechanism`, `feed-forward-sfm.coordinate-grounding`, `feed-forward-sfm.long-context-memory`.
- **Bundle truth (co_requires)**: [[anchor-frame-scale-grounding_chen2026]], [[compact-trajectory-memory-tokens_chen2026]], [[video-rope-on-trajectory-tokens_chen2026]]. The ablation shows each provides distinct value — partial adoption is a legitimate design point but is not GCA; it is a partial-GCA.

## Pipeline-shape implications

Threads adopting GCA must structure their pipeline as a coordinated trio of nodes:

- node-A (coordinate-grounding): anchor context — first $n$ frames frozen as fixed reference, anchor-point-cloud scale normalization.
- node-B (attention-mechanism): structured three-way attention mask (frame $t$'s queries → $\mathcal{A} \cup \mathcal{W} \cup \mathcal{M}$).
- node-C (long-context-memory): 6-token eviction for frames $\notin \mathcal{A} \cup \mathcal{W}$ + Video RoPE for temporal ordering.

Edges: node-A provides anchor tokens consumed by node-B; node-C emits compact trajectory tokens consumed by node-B; node-B outputs attended tokens that feed downstream camera + depth heads.

## Trade-offs vs. the decomposed pipeline

This is a topology-rewrite that *introduces* a coordinate-grounding stage, not a collapse. Trade-off analysis focuses on what GCA costs relative to simpler alternatives:

- **vs. full causal attention (Stream3R, StreamVGGT)**: GCA wins on memory ($O(1)$ per-frame growth vs. $O(T)$) and, surprisingly, on accuracy at long sequences (Table 7: bounded window-64 beats full causal on ATE + RPE-trans). Loses marginally on RPE-rot. Net win.
- **vs. RNN-style recurrence (CUT3R)**: GCA's structured three-context retention avoids CUT3R's aggressive compression (shown in ATE scaling 18→32 vs GCA 6→7). Net win.
- **vs. TTT fast-weights (TTT3R, ZipMap, LoGeR)**: GCA avoids test-time parameter updates; fully feed-forward; deterministic inference. Loses the adaptive benefit TTT provides under distribution shift — untested whether GCA generalizes as well under non-training-distribution video.
- **vs. hand-crafted SLAM backend (VGGT-SLAM, MASt3R-SLAM)**: end-to-end differentiable, no loop-closure detection (LingBot's own stated limitation).

## Open questions

- Does GCA generalize well outside stage-2's training distribution? No domain-shift robustness test in the paper.
- What happens when the anchor frames fail (static camera, near-planar geometry)? Unanswered.
- Can GCA compose with TTT inference-time adaptation (apply TTT updates to the trajectory-memory tokens)? Proposed as Bet #032 on the thread.
- Can the compact-trajectory-memory sub-idea replace CUT3R's RNN state independently of the anchor context? Proposed as Bet #033.
- Optimal $n$ (anchor count) and $k$ (window size)? Paper uses $n=3$, $k=64$ at inference; no sensitivity study.
- How much of the performance is attributable to GCA vs. the 36.9K-GPU-hour two-stage training? Without ablating stage-1 pre-training, GCA's standalone contribution is unmeasurable.

## Portability of sub-ideas

Even when a full GCA adoption is impractical, individual sub-ideas are independently useful:

- [[anchor-frame-scale-grounding_chen2026]] — drop-in scale-normalization for any monocular streaming feed-forward SfM that has a ViT tokenizer.
- [[compact-trajectory-memory-tokens_chen2026]] — replaces CUT3R's RNN state or StreamVGGT's full-history KV-cache.
- [[video-rope-on-trajectory-tokens_chen2026]] — temporal positional encoding applies to any compressed-memory scheme with latent tokens.

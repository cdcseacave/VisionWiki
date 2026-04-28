---
title: Anchor-frame scale + coordinate grounding for streaming feed-forward SfM
type: idea
source_paper: wiki/papers/chen2026_lingbot-map.md
also_in: []

scope: stage-swap
stages: [feed-forward-sfm.coordinate-grounding]

inputs: [first-n-frames-of-stream]
outputs: [anchor-point-cloud, anchor-tokens-fixed-reference, scale-factor-s, learned-anchor-token-for-identification]
assumptions: [monocular-or-multi-view, first-n-frames-have-sufficient-parallax, static-scene, up-to-scale-or-metric-when-gt-available]
requires_upstream_property: [vit-tokens-per-anchor-frame, per-frame-learnable-anchor-token]
requires_downstream_property: [attention-mechanism-can-attend-to-anchor-tokens-persistently]
learned_params: [anchor-token-embedding, attention-layers-that-condition-on-anchor-tokens]
failure_modes: [degenerate-geometry-static-first-frames, near-planar-anchor-scene, motion-blurred-anchor-frames, anchor-frames-dominated-by-single-surface]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [coordinate-grounding, scale-normalization, streaming, anchor, sfm, feed-forward]
created: 2026-04-24
updated: 2026-04-24
status: unclaimed
---

## Mechanism

Monocular reconstruction is scale-ambiguous; streaming makes scale-definition-at-start-time architecturally necessary because future frames aren't available for global normalization. DUSt3R/VGGT solve this by normalizing with respect to the *global* point cloud across all frames — incompatible with causal streaming.

LingBot's anchor context resolves this as follows:

1. Designate the first $n$ frames ($n=3$) as **anchor frames**.
2. During training, compute scale from the anchor frames' ground-truth point cloud:
   $$s = \frac{1}{|\mathcal{X}^{\text{anchor}}|} \sum_{\mathbf{x} \in \mathcal{X}^{\text{anchor}}} \|\mathbf{x}\|_2$$
   Divide all GT depths and camera translations by $s$ before computing loss. This teaches the network that the anchor-frame point cloud defines a canonical scale.
3. Augment each frame's tokens with a learnable **anchor token** $\mathbf{a} \in \mathbb{R}^C$. Apply full attention among the $n$ anchor frames' image tokens + anchor token. This identifies them and distinguishes them from subsequent streaming frames.
4. After initialization, anchor tokens (and the $n$ anchor frames' image tokens) are **retained permanently** in the attention context. Every subsequent frame attends to them as a fixed reference.

At inference, there's no GT: the network's stage-2-trained representation of the anchor tokens encodes a consistent internal scale convention, conditioned by the image content of the anchor frames.

The anchor context is a *learned* gauge-decoupling mechanism: the coordinate frame is anchored to the first $n$ frames instead of continuously updating with every new frame. Compared to re-referencing per-frame (drift) or global-pointcloud normalization (non-streaming), this is the Pareto-optimal compromise for online monocular reconstruction.

## Why it wins

Ablation (Table 6 rows 1 → 2): adding anchor initialization to the Rel-Loss-only baseline improves AUC@3 from 9.80 → 13.63 (+3.83) and ATE from 8.59 → 7.88 (−0.71). The improvement is on *both* global trajectory consistency (AUC@3) and local pose accuracy (ATE) — evidence that a well-defined coordinate reference helps register every subsequent frame, not just the global trajectory.

The effect is cleanly isolated: the only change between rows 1 and 2 is anchor init; every other component is held off. So this is the strongest available evidence for the anchor-scale mechanism alone.

## Preconditions & compatibility

- Requires a streaming-capable feed-forward SfM backbone with ViT tokenization and the ability to maintain per-frame learnable tokens (camera, register, anchor). VGGT-family architectures satisfy this.
- Requires $n$ anchor frames with non-degenerate geometry (parallax, non-planar).
- Compatible with: any downstream attention mechanism (GCA, causal, full) — the anchor is a *precondition* of the attention stage, not a specific attention pattern.
- Portable: can be adopted by models that don't use GCA. E.g., could plug into a Stream3R-style causal-attention model to provide scale grounding independently of the main attention topology.

## Trade-offs vs. alternatives

- **vs. DUSt3R pair-alignment scale**: pair-wise scale is local and drifts; anchor scale is fixed. Net win for long sequences.
- **vs. VGGT global-pointcloud normalization**: identical target (well-defined scale), but anchor-scale is *streaming-compatible*; VGGT-normalization requires all frames. Streaming use case cleanly.
- **vs. no scale normalization**: model must implicitly infer scale; unreliable at long sequences (paper's baseline row 1 confirms).
- **Cost**: requires dedicating attention capacity to the first $n$ frames forever. At $n=3$, $M=500$, that's 1500+ tokens consumed permanently from every frame's context. Manageable in GCA (which trades off via compact trajectory memory) but may be wasteful in a full-attention regime.

## Open questions

- What happens when anchor frames have no parallax (static camera at stream start)? Untested.
- How does anchor quality degrade with motion blur, low light, or occlusion?
- Is $n=3$ optimal? No sensitivity sweep reported.
- Can the anchor be *refreshed* mid-stream (e.g., on scene transitions)? Not tested.
- Does the anchor degrade with very long sequences where its relevance diminishes? 10K-frame results suggest no, but not formally measured.
- Could a "re-anchoring" mechanism (periodic anchor updates) further improve long-sequence stability? Open question.

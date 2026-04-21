---
title: Per-frame 3D cache with visibility-score geometry-aware retrieval
type: idea
source_paper: wiki/papers/shen2026_lyra2.md
also_in: []

scope: stage-swap
stages: [video-world.spatial-memory]
collapses: []
splits_into: []
rewrites: {}

inputs: [history-frames, per-frame-depth, camera-intrinsics, camera-extrinsics, target-camera]
outputs: [N-retrieved-frames, visibility-scores]
assumptions: [static-scene, known-intrinsics, mono-depth-reliable-per-frame, bounded-per-frame-pose-drift]
requires_upstream_property: [per-frame-depth-estimate, calibrated-poses]
requires_downstream_property: [consumer-accepts-bounded-n-frame-retrieval, consumer-exploits-geometric-correspondence]
learned_params: []
failure_modes: [cascading-pose-drift-corrupts-visibility-scores, high-frequency-depth-noise-breaks-occlusion-threshold, non-static-scene-invalidates-per-frame-independence, dense-revisit-patterns-exceeding-N-budget]

requires: []
unlocks: [canonical-coord-warp-injection_shen2026]
co_requires: [canonical-coord-warp-injection_shen2026]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [video-generation, long-horizon, spatial-memory, retrieval, 3d-cache, visibility-score, lyra-2, information-routing]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

Maintain a **3D cache** `C` that grows incrementally as the autoregressive video generator emits frames. For each generated frame `i` with estimated depth `D_i` (from a mono-depth predictor — DAv3 in the paper) and camera `(T_i, K_i)`, store two components:

1. **Full-resolution depth** `D_i` + camera parameters — preserved for precise correspondence computation later.
2. **Downsampled point cloud** `P_i ∈ ℝ^{(H/d)×(W/d)×3}`, obtained by subsampling `D_i` by factor `d=8` and unprojecting to world coordinates — used only for retrieval.

Critically, the cache stores **per-frame geometry independently — no fusion into a single accumulated scene**. This is the load-bearing design choice: competing approaches (global point-cloud conditioning in Gen3C/SPMem/Lyra 1) accumulate cross-view depth misalignment into a single corrupted reconstruction, which then poisons future frames.

**Visibility-score retrieval.** At each autoregressive step with target camera `(T*, K*)`:
- Project every downsampled point cloud `P_i` onto the target image plane.
- For each pixel on the target plane, take the minimum projected depth across all source frames (per-pixel occlusion handling).
- A point from frame `i` is **visible** iff its depth is within `δ=0.1` (normalized depth units) of the per-pixel minimum.
- Visibility score `φ(i) = |visible points of frame i|`.

**Training vs inference selection**:
- *Training*: sample history frames proportional to `φ(i)` — exposes the model to a wider range of retrieval results.
- *Inference*: **greedy coverage** — iteratively pick the frame that covers the most not-yet-covered target pixels, stopping at `N_s=5`. Avoids redundant selection of nearby viewpoints.

The result is that even when the camera revisits a region hundreds of frames later — far outside the model's temporal context window — retrieval can still surface the relevant observations via 3D overlap, not temporal proximity.

## Why it wins

The load-bearing ablation is Table 3 "w/ Global Point Cloud" on Tanks-and-Temples (Lyra 2.0): replacing the per-frame cache + retrieval with a single accumulated point cloud conditioning drops **Camera Controllability 63.87 → 49.86 (−14.01)** and **Style Consistency 85.07 → 82.42**. The ablation combines the loss of per-frame independence with the loss of retrieval-based selection, so the `per-frame-independence` and `retrieval-based selection` contributions are not individually isolated — but the paper's qualitative Fig. 6 shows the global-point-cloud variant producing "noticeably inaccurate camera poses," consistent with depth accumulation corrupting the conditioning signal.

**Design-rationale evidence** (from §4.2): the paper explicitly justifies per-frame independence as avoiding "accumulating cross-view misalignment from imperfect depth into a single corrupted reconstruction." This is the anti-Gen3C/SPMem argument.

Coverage analysis (Fig. 9, App. A.1): `N_s=5` — 4 frames at subsample factor 2 plus 1 at full resolution — provides the favorable trade-off between target-frame coverage and compute. `N_s=1` covers <50% of revisit targets; beyond `N_s=5` the marginal coverage gain flattens.

## Preconditions & compatibility

- **Upstream**: the autoregressive video generator must produce per-frame depth (via any mono-depth predictor — DAv3 in Lyra 2.0; any mono-depth backbone would suffice) and track camera intrinsics/extrinsics. Cached geometry uses no learned parameters of its own.
- **Downstream**: the retrieved frame set must be consumed by a backbone that can exploit multi-view geometric conditioning. In Lyra 2.0, the downstream consumer is `[[canonical-coord-warp-injection_shen2026]]` — it forward-warps canonical coordinates through the retrieved frames' depth to produce dense correspondences. **Without a correspondence-aware downstream, retrieval alone leaves the geometric relation implicit and the DiT has to re-infer it from pixel self-attention, which degrades under large viewpoint change.** This bundle constraint is why `co_requires` includes Idea 2.
- **Cross-thread transfer**: applicable to any long-horizon autoregressive generative model with a per-frame-geometry estimate — would plug into world-model backbones beyond the Wan 2.1 DiT (e.g. future video-diffusion-based 3D-scene generators).

## Trade-offs vs. the decomposed pipeline

Not applicable — `stage-swap` scope: fills `[[video-world.spatial-memory]]` as an alternative to the global-accumulated-3D-memory family (SPMem, Gen3C, Lyra 1). The stage slot itself is new (introduced with this idea page + stage page), but the pipeline topology is unchanged: one stage slot, different filler.

## Open questions

- **Robustness to depth noise over very long sequences**: the δ=0.1 threshold is fixed. Under cumulative per-frame depth drift, visibility scores for old frames may monotonically decay (all their points look "occluded" by newer, more accurate depth). The paper does not characterize how retrieval behaves at 2000+ frames; Table 1 evaluation is on frames up to ~800.
- **Non-static scenes**: the per-frame-independence invariant assumes a rigid world. Dynamic objects will produce inconsistent per-frame depth at the same spatial location; visibility scoring may then prefer recent frames by construction, collapsing back toward temporal locality. Lyra 2.0 explicitly lists "static environments" as a limitation (§6 Discussion).
- **Transfer to causal architectures**: Lyra 2.0's backbone is bi-directional DiT. The retrieval mechanism is architecture-agnostic, but its interaction with causal-KV-cache-based backbones (Self-Forcing, WorldMem) is untested — there may be redundancy with already-maintained KV state.
- **Isolation of retrieval alone** (without correspondence injection): the paper does not ablate `retrieval + RGB-conditioning` (no canonical-coord warp) as a separate variant, so we cannot cleanly attribute how much of the 14-point CC gain is from *which frames are in context* vs *how their geometry is conveyed*.

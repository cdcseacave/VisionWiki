---
title: Per-Gaussian Identity Encoding supervised by SAM masks
type: idea
source_paper: wiki/papers/ye2024_gaussian-grouping.md
also_in: []

scope: stage-swap
stages: [lifting-foundation-models.per-primitive-identity]
collapses: []
splits_into: []
rewrites: {}

inputs: [3d-gaussians, per-view-sam-masks, cross-view-mask-associations]
outputs: [per-gaussian-identity-vector, per-view-rendered-identity]
assumptions: [static-scene, posed-input, sam-masks-available-per-view]
requires_upstream_property: [sam-masks-cross-view-tracked]
requires_downstream_property: [identity-rendered-via-feature-channel]
learned_params: [gaussian.identity]
failure_modes: [long-occlusion-fragments-ids, identity-dim-is-free-hyperparam]

requires: []
unlocks: [gaussian-grouping-spatial-consistency_ye2024]
co_requires: [cross-view-mask-association-iou-similarity_ye2024]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, identity-encoding, sam-lifting, segmentation]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Each Gaussian carries a learnable low-dim identity vector `id_i`. At render time, these identities alpha-blend through the 3DGS rasterizer exactly like SH coefficients — producing a per-pixel "rendered identity" map. The training signal is cross-entropy between the rendered identity and SAM's 2D instance masks across views (after cross-view association via IoU+feature similarity gives each mask a stable track ID). The identity channel receives no direct geometric signal — it inherits geometric fidelity from the shared Gaussian positions.

## Why it wins

Supervision comes for free from SAM (no 3D labels needed); the recipe transfers from any 2D mask source. Rendering identity through the same rasterizer as color means identity and appearance share the same anti-aliasing, depth-sorting, and transparency handling. The paper shows clean per-instance edit operations (remove / inpaint / recolor / relocate) that NeRF-based predecessors could not match.

## Preconditions & compatibility

Requires a rasterizer that supports ≥1 extra feature channel (gsplat's `colors` tensor already does — see [[nerfstudio]] "feature channel trick"). Requires SAM (or any instance-mask source). Bundles with cross-view mask association (`co_requires:`) — without tracking, masks in different views get different IDs and the supervision is noise.

## Open questions

- SAM 3 ([[carion2026_sam-3]]) emits cross-view IDs natively. Does it obsolete the association module? Bet #013 tests this.
- Identity dimensionality has no principled choice.

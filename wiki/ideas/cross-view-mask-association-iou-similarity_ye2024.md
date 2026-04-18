---
title: Cross-view SAM mask association via IoU + feature similarity
type: idea
source_paper: wiki/papers/ye2024_gaussian-grouping.md
also_in: []

scope: stage-swap
stages: [lifting-foundation-models.cross-view-id-consistency]
collapses: []
splits_into: []
rewrites: {}

inputs: [per-view-sam-masks, per-view-features]
outputs: [per-mask-track-id]
assumptions: [static-scene, posed-input, moderate-viewpoint-change-per-step]
requires_upstream_property: [sam-masks-per-view-available]
requires_downstream_property: [consumer-tolerates-occasional-id-fragmentation]
learned_params: []
failure_modes: [long-occlusion-breaks-tracks, fragmented-ids-in-large-viewpoint-jumps]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: [sam3-native-video-ids_carion2026]

tags: [sam-lifting, cross-view-tracking, heuristic]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Given SAM masks in view `V_i` and `V_j`, match each mask in `V_i` to masks in `V_j` by computing (a) 2D IoU of warped masks under the estimated camera-to-camera homography, and (b) feature-similarity of average DINO features pooled within each mask. A fixed threshold on the weighted score assigns the same track ID; unmatched masks start a new track.

## Why it wins

Gives the downstream identity-encoding idea consistent supervision across views without per-scene retraining of a tracker. The paper's qualitative comparison shows clean cross-view identity in short/medium video sequences where naive per-frame segmentation would spin up a new ID per view.

## Preconditions & compatibility

Requires approximate camera poses and DINO (or similar) features per view. Breaks under large viewpoint jumps and long occlusions — an acknowledged failure mode. **Contradicted by** [[sam3-native-video-ids_carion2026]]: SAM 3 emits cross-view IDs natively and removes the need for this module entirely (Bet #013).

## Open questions

- Is there a principled failure detector that flags frames where association is unreliable so the downstream identity loss can be down-weighted?
- Can the same recipe work with non-SAM masks (e.g. instance segmentation from SAM 3)?

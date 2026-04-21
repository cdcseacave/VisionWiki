---
title: Matcher-score-based next-view selection for incremental SfM
type: idea
source_paper: wiki/papers/pataki2025_mp-sfm.md
also_in: []

scope: stage-swap
stages: [sfm.next-view-registration, sfm.next-view-scheduling]
collapses: []
splits_into: []
rewrites: {}

inputs: [candidate-view-set, feature-matcher-scores-to-registered-images]
outputs: [ranked-next-view-order]
assumptions: [deep-matcher-outputs-per-match-confidence, incremental-sfm]
requires_upstream_property: [learned-matcher-with-calibrated-per-match-scores]
requires_downstream_property: [registration-robust-to-hallucinated-inliers]
learned_params: []
failure_modes: [matcher-with-uncalibrated-scores-degrades-ordering, score-inflation-on-symmetric-content]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, next-view-selection, learned-matcher, mp-sfm]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

For each unregistered view $c$, score candidate registered images $i \in \mathcal{R}$ by $\arg\max_{i} \left( \sum q_{c,i} + \sum q^*_{c,i} \right)$ where $q_{c,i}$ are sparse-match scores and $q^*_{c,i}$ are dense-match scores produced by the learned matcher (MASt3R / RoMa / LightGlue). Rank views by this score, try registrations in order, accept the first that passes.

Contrast with two natural alternatives: (1) COLMAP's default *maximum number of inliers* (hallucinates inlier masses through symmetric surfaces when using MASt3R-class matchers); (2) *number of visible 3D points* adapted to lifted points (scales with bundle size rather than new-view quality, biasing toward late registrations). Summed matcher scores weight each potential correspondence by the matcher's own confidence, which is a better proxy for "real match" than a binary inlier flag.

## Why it wins

Ablation Table 8 (MP-SfM appendix). With MASt3R matching on ETH3D minimal-overlap:
- `num. vis. points` → AUC@1°/5°/20° = 33.9/67.2/81.2 (best non-default)
- `num. inlier corresp.` → 33.5/66.1/77.5
- **sum of matcher scores (ours)** → 39.7/67.0/78.1 on SMERF minimal; also wins on ETH3D.

Inlier-count selection fails catastrophically on symmetric SMERF scenes because MASt3R hallucinates matches through surfaces; matcher scores discount these implicitly. With SuperPoint+LightGlue (calibrated sparse matcher) the gap is smaller, confirming the win is matcher-conditional.

## Preconditions & compatibility

Requires a matcher whose per-match scores are informative (not binary). Works best with MASt3R or RoMa; works with LightGlue/SuperPoint; likely degrades with uncalibrated matchers. No downstream changes — pure ordering swap at the [[sfm.next-view-registration]] / [[sfm.next-view-scheduling]] boundary.

Composes naturally with [[mono-depth-normal-constrained-incremental-sfm_pataki2025]] (co-required in that system's actual pipeline). Independently adoptable by any incremental SfM consuming a learned matcher — e.g., candidate port into [[cusfm-slam-prior-factor-graph_yu2025]] or [[depth-constrained-jacobian_zhong2026]]'s view scheduling if they incorporate a deep matcher.

## Trade-offs vs. the decomposed pipeline

Not applicable — `stage-swap` scope. The swap preserves the existing next-view stage's interface; no pipeline shape change.

## Open questions

- How does the benefit scale with matcher calibration quality? Better-calibrated matchers (e.g., RoMa-v2 with [[roma-v2-predictive-covariance_edstedt2025]]) could amplify the gain.
- Does summing scores versus taking a max-score-per-candidate make a difference? The paper sums; unclear if max-over-candidates handles local concentrations (e.g., one very high-confidence region vs. many medium) better.
- For matchers without score outputs (e.g., pure geometric), does a proxy score (cycle-consistency, mutual-nearest-neighbor ratio) recover the benefit?

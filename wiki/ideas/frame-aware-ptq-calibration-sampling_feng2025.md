---
title: Frame-aware noise-filtered PTQ calibration sampling for VGGT
type: idea
source_paper: wiki/papers/feng2025_quantvggt.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.ptq-calibration-sampling]
collapses: []
splits_into: []
rewrites: {}

inputs: [candidate-image-sequences, trained-vggt-checkpoint]
outputs: [curated-calibration-sample-set, per-cluster-statistics]
assumptions: [multi-view-training-data-distribution-similar-to-deployment, access-to-deep-layer-activations-during-sampling]
requires_upstream_property: [converged-training-checkpoint, candidate-calibration-pool-of-representative-image-sequences]
requires_downstream_property: [ptq-transform-accepts-externally-chosen-calibration-set]
learned_params: [per-cluster-centroids, noise-threshold-percentile]
failure_modes: [small-candidate-pool-clustering-degenerates, single-image-inputs-defeat-frame-aware-premise]

requires: []
unlocks: []
co_requires: [vggt-dual-smoothed-quantization_feng2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [vggt, quantization-calibration, ptq, frame-aware-clustering, efficiency]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
---

## Mechanism

A two-part procedure for selecting calibration samples used to estimate quantization ranges during post-training quantization of VGGT. Runs as a preprocessing step before the quantization transform — see `[[feed-forward-sfm.ptq-calibration-sampling]]` stage.

**Part 1 — Noise filtering via deep-layer statistics.** For each candidate calibration sequence in the pool, run a forward pass and record the activation statistics at a deep layer (paper uses the penultimate aggregator block). Samples whose activations fall in the tails — top/bottom `p%` of deep-layer max-abs values, with `p ≈ 5` — are discarded as "noisy extremes" (textureless walls, degenerate viewpoints, etc.). These tail samples are overrepresentative of rare input distributions and their calibration contribution skews the quantization range estimate.

**Part 2 — Frame-aware diverse clustering.** The remaining filtered samples are clustered using features derived from *per-frame* activation distributions (not per-image features). Specifically, each sample's feature vector concatenates mean + variance + quantile statistics of activations computed across the frames of that multi-view sample. K-means on these frame-distribution features yields clusters that group calibration samples by *multi-view distributional similarity* rather than by image-level appearance.

The final calibration set draws uniformly across clusters, ensuring coverage of the diverse multi-view regimes VGGT was trained on (object-centric sparse-view, outdoor dense-view, textureless indoor, etc.) rather than overfitting the quantization ranges to any single regime.

Total calibration pool size: ~128–256 samples from the curated clusters, consistent with PTQ literature for billion-scale transformers.

## Why it wins

Direct isolation via Table 10 (paper §4.3): swapping random calibration sampling for NFDS improves W4A4 AUC@30 by 2.3 points on CO3D, 3.1 on ScanNet, and *stabilizes* the W4A4 result across random seeds. The seed-variance reduction is the under-appreciated win — PTQ literature often reports best-of-N seeds; NFDS makes first-seed results match best-seed results.

Motivation (paper Fig. 4): label-based clustering mixes textureless-wall samples with object-centric samples; direct feature-based clustering clusters by appearance (SIFT-like image similarity) and misses the multi-view distribution structure; frame-aware clustering recovers clusters that correspond to VGGT's training regimes.

The deep-layer noise filter (Part 1) is the cheaper half — ~5 AUC-point improvement on its own. The frame-aware clustering (Part 2) adds another 2–3 AUC points on top. Neither alone reaches the full W4A4 headline.

## Preconditions & compatibility

Consumes: a candidate pool of multi-view image sequences (ideally from the VGGT training distribution) + the trained checkpoint. Produces: a curated calibration sample set that the `[[feed-forward-sfm.numerical-precision]]` stage's filler consumes.

**Bundle constraint (`co_requires:`)**: [[vggt-dual-smoothed-quantization_feng2025]] — practically inseparable from DSFQ in the paper's headline W4A4 result. However, the calibration procedure is mechanism-agnostic in principle: it could feed any PTQ transform (SmoothQuant, QuaRot, SpinQuant). Transfer to non-DSFQ quantization is an open question.

Composition edges:
- **`co_requires: [vggt-dual-smoothed-quantization_feng2025]`** — bundle.
- **Independent of** FastVGGT / SparseVGGT — calibration sampling happens offline before inference-time attention optimizations.

## Pipeline-shape implications

`scope: drop-in` on the new [[feed-forward-sfm.ptq-calibration-sampling]] stage. No topology change; introduces a pre-quantization calibration node that runs once at deployment.

## Trade-offs vs. the decomposed pipeline

Not applicable. Trade-off vs random calibration sampling: requires a forward-pass statistics sweep on candidate sequences (amortized across many deployments — the calibration set can be reused across backbone revisions). Single-image deployment inputs defeat the frame-aware premise (no multi-view frames means frame-distribution features degenerate to per-image features).

## Open questions

- **Transfer to non-VGGT PTQ pipelines**: the frame-aware clustering premise assumes multi-view-frame-structured inputs. Does it help for monocular-depth-foundation-model PTQ (e.g., MoGe, Depth Anything) where there are no "frames per sample"?
- **Interaction with W2A2 or W3A3**: at lower bit-widths, calibration range estimates become more sensitive. Does NFDS's seed-variance reduction hold, or do the rare clusters dominate?
- **Online calibration**: paper uses offline one-shot calibration. Can the procedure be adapted to a streaming deployment where the calibration pool changes over time (concept drift)?
- **Minimum calibration pool size**: paper uses O(10,000) candidate samples to filter + cluster. What's the floor below which the clustering degenerates?

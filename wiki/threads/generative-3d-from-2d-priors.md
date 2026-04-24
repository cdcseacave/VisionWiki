---
title: Generative 3D from 2D Priors
type: thread
tags: [generative-3d, single-image, flow-matching, sam-3d, latent-diffusion, video-world, explorable-3d, lyra-2]
created: 2026-04-18
updated: 2026-04-24
sources: [wiki/papers/chen2025_sam-3d.md, wiki/papers/wang2026_feed-forward-3d-scene-modeling.md, wiki/papers/shen2026_lyra2.md, wiki/papers/bahmani2025_lyra.md, wiki/papers/meng2026_seen2scene.md]
operating_points: [op:default, op:explorable-scene]
status: draft
---

## Goal

Produce 3D scene geometry + texture + per-object layout from severely under-constrained input (single RGB image, or image + minimal prompts) by combining learned 2D/3D priors with generative modeling — rather than reconstructing from multi-view photometric consistency. Success metric is human-preference fidelity and downstream usability (asset creation, sparse-view reconstruction priors), not photometric accuracy against a specific capture.

## Goal contract (optional, structured)

```yaml
metric: [human-preference-rate-vs-baseline, per-object-geometry-quality, scene-layout-plausibility]
target_regime: [single-image-input, cluttered-natural-image, static-scene]
constraints: [per-scene-optimization-acceptable, commercial-license-tracked-per-paper]
required_capabilities: [per-object-shape-generation, per-object-texture-generation, scene-layout-prediction, real-image-domain-coverage]
```

## SOTA pipelines

The thread now tracks two operating points: `op:default` (single-image to per-object 3D scene — the 3D-world paradigm) and `op:explorable-scene` (single-image to large-scale explorable 3D walkthrough — the video-world paradigm). The split was triggered by [[shen2026_lyra2|Lyra 2.0]] ingest (2026-04-21), which occupies a fundamentally different target regime (scene-scale, long-horizon, video-mediated) than SAM 3D's object-scale 3D-latent-flow-matching. The [[information-routing-vs-3d-rendering-memory]] concept page articulates the video-world-internal design-principle axis for `op:explorable-scene`.

### op:default (single-image to per-object 3D scene; 3D-world paradigm)

- **n1** · stage(s): `[[generative-3d.shape-generation]]` · filler: [[two-stage-latent-flow-matching-scene_chen2025]]. Source: [SAM 3D (Chen 2025)](wiki/papers/chen2025_sam-3d.md). Per-object latent → decoded geometry + texture via flow-matching. Upstream: single RGB image. Downstream: [n2].
- **n2** · stage(s): `[[generative-3d.scene-layout]]` · filler: [[two-stage-latent-flow-matching-scene_chen2025]] (layout module — shared idea across stages, hence composite). Predicts per-object placement (pose + scale). Upstream: [n1].
- **n3** · stage(s): `[[generative-3d.training-data-curation]]` · (training-time, not inference) · filler: [[real-image-data-engine_chen2025]]. Human/model-in-the-loop data engine producing real-image 3D annotations that unlock generalization. Bundle constraint: `co_requires` the data engine.

### op:explorable-scene (single-image to long-horizon explorable 3D walkthrough; video-world paradigm)

Single input image + interactive camera trajectory → autoregressively generated walkthrough video → feed-forward lift to 3DGS + mesh. All filler ideas from [[shen2026_lyra2]] (2026-04).

- **n4** · stage(s): `[[video-world.spatial-memory]]` · filler: [[per-frame-3d-cache-retrieval_shen2026]]. Per-frame 3D cache (full-res depth + downsampled point cloud at `d=8`) + visibility-score greedy-coverage retrieval (`N_s=5`). No global fusion. Source: [Shen et al. 2026](wiki/papers/shen2026_lyra2.md). Gain over prior: Table 3 "w/ Global Point Cloud" row — **+14.01 Camera Controllability** on T&T vs the global-memory alternative (bundle with n5). Upstream: single RGB image + iterative prior-chunk generations (n7 output). Downstream: [n5].
- **n5** · stage(s): `[[video-world.correspondence-injection]]` · filler: [[canonical-coord-warp-injection_shen2026]]. Forward-warped 4-channel `[Ĉ_j; D̂_j]` canonical coordinate maps encoded via MLP + positional encoding, injected on Q/K only of DiT self-attention, zero-init aggregator. Source: [Shen et al. 2026](wiki/papers/shen2026_lyra2.md). Gain over prior: Table 3 "w/ Explicit Corr. Fusion" — +6.58 Camera Controllability vs hard depth-reasoning fusion. **Bundled with n4** — both must be adopted together. Upstream: [n4]. Downstream: [n6].
- **n6** · stage(s): `[[video-world.drift-mitigation-training]]` · (training-time) · filler: [[self-augmented-history-training_shen2026]]. One-step flow-matching noise injection on encoded history latent with `p_aug=0.7`, `t ~ U(0, 0.5)`. Source: [Shen et al. 2026](wiki/papers/shen2026_lyra2.md). Gain over prior: Table 3 "w/o Self-Augmentation" — +7.09 Style Consistency, +9.95 Camera Controllability in isolation; ~35× cheaper than Self-Forcing on bi-directional DiTs. Upstream: clean GT training video + [n4, n5] conditioning context. Downstream: [n7, n10].
- **n7** · stage(s): `radiance-fields.rendering` (at training time, as video prediction into latent space) · filler: **Wan 2.1-14B DiT + FramePack temporal compression** (backbone; not a first-class idea — established prior art [107] + [135]). Rectified flow matching at 832×480 latent, causal VAE 8×/4×, FlowUniPC 35-step inference. Upstream: [n4, n5, n6]. Downstream: [n8].
- **n8** · stage(s): `[[generative-3d.shape-generation]]` · filler: [[downsampled-gaussian-dpt-head_shen2026]] (+ fine-tune on generated data per Lyra 1 precedent). DAv3 ([[depthanythingv3]]) with `k=2` strided Gaussian DPT head (4× fewer Gaussians) fine-tuned on 3K autoregressive Lyra-generated DL3DV scenes. Source: [Shen et al. 2026](wiki/papers/shen2026_lyra2.md). Gain over prior: Table 2 "Ours Full" vs "Ours + DAv3" — LPIPS-G 0.648 → 0.629 on T&T; combined with upstream video-model improvements, is the SOTA generative-reconstruction 3DGS. Upstream: [n7]. Downstream: [n9].
- **n9** · stage(s): `[[mesh-reconstruction.extraction]]` · filler: [[hierarchical-sparse-grid-mesh-extraction_shen2026]]. OpenVDB adaptive-level sparse grid + SDF from 3DGS depth + per-level marching cubes + level-transition merge. Source: [Shen et al. 2026](wiki/papers/shen2026_lyra2.md). Gain over prior: no quantitative ablation — qualitative evidence via Isaac Sim integration (Fig. 7). **Evidence weaker — flagged on the idea page.** Upstream: [n8]. Downstream: 3D applications (simulation, VR).
- **n10** · stage(s): `[[video-world.distillation]]` · (deployment-time) · filler: [[dmd-with-self-aug_shen2026]]. DMD 35→4 step + CFG distilled + self-aug retained during distillation. Source: [Shen et al. 2026](wiki/papers/shen2026_lyra2.md). Gain over prior: ~13× per-step speedup (194s → 15s per 80-frame step on GB200). **Bundled with n6** — distillation retention without teacher-side augmentation is vacuous. Upstream: teacher model (n4+n5+n6+n7). Downstream: fast-student inference graph.

**Node bundle note**: n4 ↔ n5 is a hard co-require (canonical-coord injection needs per-frame retrieved geometry; retrieval without dense correspondence leaves the DiT to infer geometry from pixel attention alone). n6 ↔ n10 is also a hard co-require (distillation retention of a property the teacher lacks is vacuous).

## Pipeline lineage

- 2025 · op:default · initial pipeline: two-stage latent flow-matching + data engine. Driver: [[chen2025_sam-3d]].
- 2026-04-21 · op:explorable-scene · **new OP (topology-rewrite)** · scope: new-paradigm · nodes introduced: [n4, n5, n6, n7, n8, n9, n10]. Driver: [[shen2026_lyra2]]. Rationale: Wang 2026 §7.4 named video-world-vs-3D-world as a capability gap on both this thread and [[radiance-field-evolution]]; Lyra 2.0 is the first in-wiki paper committed to the video-world side and fills the gap end-to-end from single image to 3DGS + mesh. Placed as an additional OP (now 2 OPs, under the ≤3 cap) rather than forking a sibling thread because the **input regime is identical** to op:default (single image → 3D scene) — the difference is target scale (object vs explorable environment) and intermediate representation (3D-latent vs video → lifted 3D). Keeping both OPs on the same thread makes cross-paradigm bets (Bet #024, Bet #025, Bet #026 below) legible as thread-local graph queries rather than cross-thread.

## Candidate components / not yet integrated

- **SAM 3 masks as per-object conditioning** ([[carion2026_sam-3]]) — the naming of SAM 3D suggests future composition; the paper doesn't yet demonstrate it. Replaces/augments: arbitrary per-object segmentation from the input image. OP considered: `op:default`.
- **DINOv3 as the image-feature frontend** ([[simeoni2025_dinov3]]) replacing whatever frozen backbone SAM 3D uses — unified feature stack across the generative + reconstructive threads. OP considered: `op:default`.
- **DUSt3R/VGGT as a reconstructive prior** for the shape generator — when a second view is available, condition on MASt3R features rather than generating freely. OP considered: `op:default`.
- **Global-3D-memory alternative** (Gen3C, SPMem, Lyra 1 family) — did not make `op:explorable-scene` because Lyra 2.0 Table 3 "w/ Global Point Cloud" row shows it loses 14.01 Camera Controllability on T&T to the per-frame-routing approach ([[per-frame-3d-cache-retrieval_shen2026]] + [[canonical-coord-warp-injection_shen2026]]). Revivable if a paper closes the error-amplification problem of accumulated depth. See [[information-routing-vs-3d-rendering-memory]] for the framing.
- **FramePack-only memory (Yume-1.5 style, no spatial memory)** — did not make `op:explorable-scene`; Table 1 shows pure-temporal-context methods (Yume-1.5, HY-WorldPlay) collapse on Camera Controllability at long horizons. Revivable only if context compression can be pushed far enough to make retrieval redundant — no current evidence this is feasible.
- **Self-Forcing on causal DiT backbones** — an alternative to [[self-augmented-history-training_shen2026]]. Loses on cost (~35× more expensive on bi-directional DiTs; comparable on causal). OP considered: `op:explorable-scene`. Revivable on causal architectures where compute is not prohibitive.
- **Video-to-3DGS without per-scene fine-tuning** — the current `op:explorable-scene` n8 filler fine-tunes DAv3 on Lyra-generated data. An off-the-shelf DAv3 is 12–20% worse on LPIPS-G (Table 2 "Ours + DAv3" vs "Ours Full"). Revivable if a DAv3 successor is robust enough to generative artifacts without fine-tune — candidate for the DAv3 paper ingest.

## Open questions & synthesis bets

### Bet #022 — SAM 3 masks as per-object conditioning into SAM 3D
status: proposed
combines: [[two-stage-latent-flow-matching-scene_chen2025]], [[sam3-native-video-ids_carion2026]]
stage_target: generative-3d.shape-generation
op_target: op:default
confidence: high
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: SAM 3 already produces per-concept masks + instance IDs; routing these into SAM 3D's per-object shape generator as explicit conditioning should make the "which things to generate" decision explicit rather than emergent, improving cluttered-scene performance.
expected_gain: Cleaner object-boundary generation; measurable human-preference lift on scenes with 5+ objects where the baseline SAM 3D over/under-segments.
risk: SAM 3 masks are 2D; lifting to 3D inherits the single-image-ambiguity problem. The two models' data distributions may not align.
validating_experiment: Inject SAM 3 masks as Stage-1 conditioning; compare vs. baseline SAM 3D on a cluttered-scene QA benchmark.
triggers: [ingest-of-idea:per-object-conditioned-3d-generation]
created: 2026-04-18 · updated: 2026-04-18

### Bet #023 — Generative 3D as a sparse-view prior for reconstruction
status: proposed
combines: [[two-stage-latent-flow-matching-scene_chen2025]], [[per-gaussian-self-supervised-confidence_radl2026]]
stage_target: radiance-fields.initialization
op_target: op:default (and cross-thread to radiance-field-evolution op:quality-per-scene)
confidence: low
magnitude: paradigm
cost: months
breakage_risk: high
hypothesis: Use SAM 3D's single-image generative output as an *initialization* for a sparse-view 3DGS reconstruction (1-2 views). CoMe-style confidence gates where the generative prior contributes vs. the photometric loss takes over. Could make sparse-view 3DGS work in regimes where current methods (pixelSplat, MVSplat) fail.
expected_gain: Sparse-view 3DGS benchmarks (1-3 views) — first method to combine a generative 3D prior with confidence-weighted photometric fitting.
risk: Generative priors at single-image fidelity may poison the photometric loss on conflicts; the interaction is untested.
validating_experiment: Hybrid pipeline on DL3DV sparse-view subset.
triggers: [ingest-of-idea:generative-reconstruction-bridge]
created: 2026-04-18 · updated: 2026-04-18

### Bet #024 — Self-augmented training as a causal-backbone Self-Forcing replacement
status: proposed
combines: [[self-augmented-history-training_shen2026]], (any-causal-autoregressive-video-generator)
stage_target: video-world.drift-mitigation-training
op_target: op:explorable-scene (and any future causal-backbone op)
confidence: high
magnitude: substantial
cost: days
breakage_risk: low
hypothesis: [[self-augmented-history-training_shen2026]] is architecture-agnostic — one-step flow-matching noise on clean-latent history is a cheap proxy for autoregressive drift that does not depend on whether the generator is bi-directional (Lyra 2.0's case) or causal (Self-Forcing's case). Applying it to a causal video-diffusion backbone where Self-Forcing was the established recipe should give Self-Forcing-like drift robustness at ~35× lower training cost per step.
expected_gain: On a causal-backbone long-horizon benchmark, match Self-Forcing's drift-robustness metrics at training cost equivalent to vanilla clean-GT-history training. Key validation: Style Consistency + Camera Controllability at 800+ frame horizons.
risk: Flow-matching schedule may not transfer cleanly to diffusion-only causal backbones — the one-step noise construction uses `(1-t)z_0 + tε` which is flow-matching-specific; an equivalent one-step corruption on a pure DDPM schedule needs adaptation. The `(p_aug, t-range)` schedule may also differ optimally across architectures.
validating_experiment: Drop self-augmentation (with flow-matching-analog noise schedule) into a causal video-diffusion backbone (e.g. a WorldMem or Self-Forcing baseline) in place of Self-Forcing; ablate on long-horizon WorldScore metrics. Compare training cost + final drift robustness.
triggers: [ingest-of-paper:self-forcing-huang-2025, ingest-of-paper:worldmem-118, ingest-of-any-causal-autoregressive-video-generator]
created: 2026-04-21 · updated: 2026-04-21

### Bet #025 — Lyra 2.0 video-world output as multi-view expansion prior for op:default SAM 3D
status: proposed
combines: [[per-frame-3d-cache-retrieval_shen2026]], [[canonical-coord-warp-injection_shen2026]], [[self-augmented-history-training_shen2026]], [[two-stage-latent-flow-matching-scene_chen2025]]
stage_target: generative-3d.shape-generation
op_target: op:default (cross-OP composition from op:explorable-scene)
confidence: low
magnitude: paradigm
cost: months
breakage_risk: high
hypothesis: SAM 3D's `op:default` shape generator struggles under ambiguous single-image input (self-occluded objects, sparse visibility). Lyra 2.0's video-world pipeline (op:explorable-scene) can convert a single input image into an arbitrarily long set of camera-controlled walkthroughs with per-frame depth. Feeding those generated multi-views (not real captures) to SAM 3D as effectively-posed input — using generated geometry from [n4/n5/n8] as the depth/pose signal — might close the single-image-ambiguity failure mode without requiring real multi-view capture. This is the mirror-image architecture of Bet #023 (generation-first, then reconstruction) applied to this thread's two OPs.
expected_gain: SAM 3D cluttered-scene performance on scenes with 5+ partially-occluded objects where single-image baseline fails; closes the thread's "Bridge to multi-view reconstruction" capability gap.
risk: Generated "multi-views" are not real — downstream SAM 3D may inherit the generator's biases + photometric inconsistencies (Lyra 2.0's own Discussion §6 flags exposure variation artifacts from DL3DV training). The combined pipeline is also the slowest configuration — single-image generation → video-world gen → feed-forward reconstruction → SAM-3D-on-feed-forward-output. Latency may be prohibitive for user-facing applications.
validating_experiment: Pipeline: SAM 3D (single image → object proposal) vs SAM 3D fed Lyra-2.0-generated multi-views of the same scene. Measure on cluttered-scene QA benchmarks; ablate number of generated views.
triggers: [ingest-of-paper:video-world-to-3d-world-bridge, real-world-deployment-latency-acceptable]
created: 2026-04-21 · updated: 2026-04-21

### Bet #026 — Canonical-coordinate injection recipe transferred to SAM 3D latent flow-matching
status: proposed
combines: [[canonical-coord-warp-injection_shen2026]], [[two-stage-latent-flow-matching-scene_chen2025]]
stage_target: generative-3d.shape-generation
op_target: op:default
confidence: med
magnitude: incremental
cost: weeks
breakage_risk: low
hypothesis: The Q/K-only zero-init injection recipe from [[canonical-coord-warp-injection_shen2026]] is a general "bolt structured guidance onto a pretrained foundation model without destabilizing it" pattern. SAM 3D's two-stage latent flow-matching is also a transformer with attention blocks; injecting scene-layout-module (n2) canonical coordinates as correspondence signals into the shape-generator (n1) on Q/K could improve cross-object placement consistency in cluttered scenes, giving the shape generator access to spatial-layout information it currently only sees through the latent conditioning.
expected_gain: Modest improvement in per-object position/orientation consistency on multi-object scenes; mostly valuable on scenes where objects occlude each other.
risk: SAM 3D's flow-matching conditioning may already carry the layout signal implicitly through the shared decoder — injection may be redundant. Also, canonical-coords need a depth source; SAM 3D doesn't have a native per-object depth in single-image inference, requiring an external mono-depth predictor.
validating_experiment: Hybrid pipeline injection of warped canonical coords (from mono-depth + SAM 3D's scene layout) into SAM 3D's shape-generation transformer Q/K; ablate on human-preference vs baseline SAM 3D on cluttered-scene QA.
triggers: [ingest-of-paper:sam-3d-architecture-details-needed-for-injection-point]
created: 2026-04-21 · updated: 2026-04-21

### Bet #027 — Lyra-2.0 video-world geometry → Seen2Scene completion (cross-thread to 3d-scene-completion)
status: proposed
combines: [[per-frame-3d-cache-retrieval_shen2026]], [[downsampled-gaussian-dpt-head_shen2026]], [[controlnet-frozen-flow-self-supervised-completion_meng2026]], [[visibility-guided-masked-flow-matching_meng2026]]
stage_target: scene-completion.condition-injection (cross-thread)
op_target: op:explorable-scene (cross-thread to 3d-scene-completion:op:default)
confidence: low
magnitude: paradigm
cost: months
breakage_risk: high
hypothesis: Lyra 2.0 generates an explorable 3DGS world from a single image, but the resulting geometry only covers regions the generated camera trajectory actually visited — the rest of the scene remains uncovered, and the geometry that *is* generated exhibits artifacts (DL3DV exposure variation propagation per Lyra 2.0 §6). Lifting the Lyra-2.0 3DGS to a partial TSDF (via per-Gaussian depth fusion with explicit visibility tracking from the generated trajectory) and running [[meng2026_seen2scene|Seen2Scene]] completion would (a) repair occluded regions the trajectory never saw, and (b) produce a globally coherent volumetric output that downstream applications can consume as a TSDF/mesh. This bridges video-world `op:explorable-scene` to a scene-completion `op:default`-style geometric output, closing the trajectory-coverage gap with a generative completion prior rather than re-generating new trajectories to fill gaps. Mirror-image to Bet #023 (generation-first then reconstruction); also a cross-thread test of how Seen2Scene's indoor-trained generative prior handles non-sensor (model-hallucinated) partial input. Cross-thread bet — primary owner [[3d-scene-completion]] Bet #003.
expected_gain: Combined pipeline produces an *explorable + complete* 3D scene from a single image; closes the "Bridge to multi-view reconstruction" capability gap. Quantifiable on a single-image-to-scene benchmark by completion fidelity at unseen viewpoints + geometric coherence (no holes in completed regions).
risk: Lyra 2.0's 3DGS uses its own coordinate frame + photometric biases; fusing to partial TSDF then completing introduces compound failure modes that neither model was designed to handle. Latency is also a concern — single-image → video-world generation → 3DGS extract → TSDF fuse → Seen2Scene complete is the slowest possible inference path. Seen2Scene is indoor-only; outdoor exploration regimes are blocked until an outdoor-trained successor exists. The visibility-aware encoder may be confused by Lyra-derived TSDFs (depth from a generative model is structurally different from sensor-grade depth).
validating_experiment: Pipeline: held-out single-image → Lyra 2.0 op:explorable-scene end-to-end → fuse the resulting 3DGS to partial TSDF with visibility tracking from the generated camera trajectory → run Seen2Scene completion → compare geometric completeness + viewpoint-novel-PSNR/LPIPS to (a) Lyra-only output and (b) Seen2Scene-on-mono-depth-fusion-of-same-image (Bet #002 of 3d-scene-completion).
triggers: [ingest-of-paper:gaussian-to-tsdf-with-visibility, real-world-deployment-latency-acceptable, ingest-of-paper:outdoor-trained-seen2scene-successor]
created: 2026-04-24 · updated: 2026-04-24

## Capability gaps

- **Reconstruction-vs-generation disentanglement** — no principled way to know when the output is reconstructed from the input image vs. hallucinated. Would unlock: credibility for downstream asset creation. Search target: uncertainty estimation in flow-matching / diffusion 3D generators. **Reinforced by [[wang2026_feed-forward-3d-scene-modeling]] §7.6**: the second feed-forward-3D survey names this explicitly as the "reconstruction-vs-generation spectrum" — a continuum, not a dichotomy, with the design question being *where* on the spectrum a method should sit for a given deployment. Confirms this gap is real and cross-thread.
- **Compositing with existing scene** — generated objects need lighting + shadow consistency with their host scene. Would unlock: scene editing pipelines. Search target: relighting-aware 3D generation.
- **Bridge to multi-view reconstruction** — when 2+ views are available, the generative prior should yield gracefully to photometric fitting. Would unlock: unified generative-reconstructive pipeline. Search target: Bet #023's hypothesis becoming a paper. [[wang2026_feed-forward-3d-scene-modeling]] §4.4.2 (Visual Augmentation) documents an adjacent pattern — *post-hoc* generative polish applied after feed-forward reconstruction to fix artifacts / hallucinate occluded regions — which is the mirror-image architecture of Bet #023 (generation-first, reconstruction-second vs. reconstruction-first, generation-second). **Partially addressed by Bet #027** (2026-04-24) which adapts a *3D-to-3D* generative completion prior ([[meng2026_seen2scene]]) as a post-processor for video-world-derived geometry — a different mirror-image specifically for `op:explorable-scene`'s coverage-gap problem.
- ~~**Video-world vs 3D-world paradigm choice**~~ **Closed 2026-04-21** by [[shen2026_lyra2|Lyra 2.0]] ingest. The thread now tracks both paradigms as separate OPs: `op:default` (3D-world: SAM 3D) and `op:explorable-scene` (video-world: Lyra 2.0). The internal design-principle axis *within* the video-world paradigm (rendering-memory vs routing) is captured as [[information-routing-vs-3d-rendering-memory]] concept page. What was a search target has become a cross-paradigm composition question (Bet #025).
- **Rendering-memory video-world variants not yet in wiki** — Gen3C, SPMem, WorldMem, Context-as-Memory, VMem all belong to the global-3D-memory family that Lyra 2.0 argues against. None are ingested yet. Would unlock: quantitative confirmation of Lyra 2.0's Table 3 ablation against in-wiki baselines, and a competing family to track under `op:explorable-scene` if the routing vs rendering choice shifts for larger-scene or denser-view regimes. Search target: Gen3C (the strongest baseline per Table 1 camera controllability) or SPMem (CaM-strongest on quality metrics).
- **Long-horizon evaluation protocol** — Lyra 2.0 reports at ~800 frames; there is no in-wiki benchmark past that horizon. Scene exploration tools will routinely exceed thousands of frames. Search target: a benchmark paper that specifically tests consistency at 2K+ frames with revisits.
- **Dynamic-scene generalization of the video-world pipeline** — Lyra 2.0 §6 Discussion explicitly lists "static environments" as a limitation. The [[radiance-field-evolution]] thread's open question on the 4D-Gaussian feed-forward cluster (4DGT, L4GM, StreamSplat, DGS-LRM) is the natural counterpart. Would unlock: dynamic-scene `op:explorable-scene`. Search target: a dynamic-scene variant of the video-world architecture, or a reconstruction-side filler for n8 that handles per-frame temporal inconsistency.
- **Photometric stability within the video model** — Lyra 2.0 §6 notes DL3DV's exposure variation propagates through training into generated videos, degrading feed-forward 3DGS at stage n8. Would unlock: cleaner lift-to-3DGS without photometric post-processing. Search target: papers training video-world models with photometric constraints or on photometrically-consistent synthetic data.

## Contradictions & tensions

- **Reconstruction vs. generation paradigm**: this thread's generative approach is orthogonal to every paper in [[radiance-field-evolution]] and [[feed-forward-structure-from-motion]], which rely on multi-view photometric consistency. No paper explicitly contrasts the two regimes on the same benchmark. [[wang2026_feed-forward-3d-scene-modeling]] reinforces rather than resolves this — its §7.6 frames the choice as a continuum to be navigated per deployment, not an either/or. The cross-thread reconstruction-vs-generation question is now an open question on [[feed-forward-structure-from-motion]] as well.

## Shelved bets / known non-compositions

(none yet)

## Sources

- [chen2025_sam-3d.md](../papers/chen2025_sam-3d.md) — op:default driver.
- [wang2026_feed-forward-3d-scene-modeling.md](../papers/wang2026_feed-forward-3d-scene-modeling.md) — second feed-forward-3D survey; §4.4.2 Visual Augmentation and §7.4 World Models + §7.6 reconstruction-vs-generation spectrum land on this thread. Named the video-world-vs-3D-world paradigm gap that Lyra 2.0 closed.
- [shen2026_lyra2.md](../papers/shen2026_lyra2.md) — **op:explorable-scene driver** (2026-04). Canonical video-world video-mediated scene generation; single-image to long-horizon 3DGS + mesh via retrieval-based memory + self-augmented training + DMD-distilled student. Drove the op-split and filed Bets #024–#026.
- [bahmani2025_lyra.md](../papers/bahmani2025_lyra.md) — **Lyra 1 stub** (2025-09). Direct precedent to Lyra 2.0; establishes the generative-reconstruction paradigm and the fine-tune-reconstructor-on-generated-data recipe that Lyra 2.0 inherits.

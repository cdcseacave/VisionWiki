---
title: DMD distillation with retained self-augmentation
type: idea
source_paper: wiki/papers/shen2026_lyra2.md
also_in: []

scope: drop-in
stages: [video-world.distillation]
collapses: []
splits_into: []
rewrites: {}

inputs: [self-augmented-teacher-model, training-distribution]
outputs: [4-step-student-model-with-cfg-distilled, drift-robust-at-inference]
assumptions: [teacher-trained-with-self-augmentation, dmd-distillation-applicable, cfg-distillable]
requires_upstream_property: [teacher-has-drift-mitigation-property]
requires_downstream_property: [none]
learned_params: [student-dit-weights-from-distillation]
failure_modes: [student-camera-controllability-degrades-with-step-reduction, distillation-without-retained-self-aug-produces-brittle-student]

requires: [self-augmented-history-training_shen2026]
unlocks: []
co_requires: [self-augmented-history-training_shen2026]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [distillation, dmd, video-generation, cfg-free, lyra-2, step-reduction, fast-inference]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

Standard Distribution Matching Distillation (DMD) [126] distills a slow teacher diffusion model into a faster student. Lyra 2.0's contribution is not DMD itself but **the observation that distillation will silently drop the drift-robustness property** acquired by training with `[[self-augmented-history-training_shen2026]]` — unless that augmentation is explicitly retained inside the distillation loop.

**The recipe**:
1. Start from the trained teacher (35 denoising steps, classifier-free guidance scale 5.0).
2. Distill to a student that generates videos in **4 denoising steps** instead of 35.
3. **Also distill the classifier-free guidance into the student** — eliminating the need for separate conditional + unconditional forward passes at inference, giving a further speedup from single-pass inference.
4. **During distillation, retain the self-augmentation strategy**: sample corrupted-history contexts for the student the same way the teacher was trained on them, so the student learns to handle inference-time degraded history — not just the teacher's clean-output distribution.

**Net speedup**: ~13× per-step generation time reduction (35 steps + 2× CFG passes → 4 steps + 1× pass). Per Appendix A.3: 194s → 15s per autoregressive step (80 frames) on a single NVIDIA GB200, including depth estimation + spatial memory retrieval + DiT denoising.

## Why it wins

**Step-reduction efficacy** (Table 1, "Ours DMD" row on Tanks-and-Temples):
- LPIPS 0.552 → 0.545 (slightly *improves*)
- FID 51.33 → 49.71 (slightly improves)
- Subjective Quality 43.35 → 43.02 (parity)
- Camera Controllability 63.87 → **58.12 (−5.75)** — moderate degradation from step reduction
- Style Consistency 85.07 → 78.91 (−6.16)

The drop in Camera Controllability + Style Consistency is the cost of fewer denoising steps — but these numbers are still above all non-Lyra baselines in Table 1 (nearest competitor SPMem: CC 45.07, SC 79.68), so the distilled student remains SOTA overall while running ~13× faster per step.

**No published ablation of "DMD without retained self-aug"**. The paper explicitly states the retention strategy is important — but does not present a side-by-side of distilled-with-aug vs distilled-without-aug. Evidence for the "retention matters" claim is theoretical (distillation smoothing effect on learned noise distributions) rather than empirical in this paper.

## Preconditions & compatibility

- **Hard bundle with `[[self-augmented-history-training_shen2026]]`**: the `requires` and `co_requires` both point there. Retaining a property you don't have is vacuous. If the teacher is not self-augmented, this idea collapses to vanilla DMD + CFG distillation.
- **Teacher architecture**: DMD requires a teacher with a known score function / denoising trajectory. Applies to flow-matching (Lyra) or diffusion backbones.
- **Bundle with retrieval + correspondence injection** (Ideas 1 + 2): inherited from the teacher; student reuses the teacher's memory mechanism unchanged.

## Trade-offs vs. the decomposed pipeline

Not applicable — `scope: drop-in`.

## Open questions

- **Empirical isolation of the "retention" claim**: the paper asserts that retaining self-augmentation during distillation preserves drift robustness. A direct ablation (distill-with-aug vs distill-without-aug, measure drift on 800+ frame horizons) would strengthen the claim. Not presented.
- **Step-count frontier**: 4 steps is chosen; 2-step or 1-step distillation (as in SDXL Turbo-class recipes) is untested for Lyra. Whether the trade-off curve stays favorable at more aggressive step reduction is open.
- **CFG distillation vs retention**: the student drops CFG entirely, saving the 2× forward pass. This removes a controllability lever (you cannot dial up CFG at inference on the distilled student). For interactive use where a user may want stronger text-prompt adherence, this is a trade-off.
- **Transferability**: the "retain the training-time augmentation inside distillation" pattern may generalize to any distillation where the teacher has learned a non-obvious robustness. Candidate recipe for any diffusion-teacher-distillation pipeline — worth articulating as a concept if a second paper validates it.

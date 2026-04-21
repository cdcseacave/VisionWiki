---
title: Video-world distillation
type: stage
slug: video-world.distillation
consumes: [trained-teacher-model, distillation-data]
produces: [fast-student-model]
invariants: [student-inherits-teacher-drift-properties-iff-distillation-preserves-them]
provides_properties: [reduced-denoising-steps, optionally-cfg-free-inference]
requires_upstream_properties: [drift-mitigation-property-of-teacher, fully-trained-teacher]
data_regime: [same-as-teacher-training-distribution, student-reuses-autoregressive-protocol]
tags: [video-generation, distillation, dmd, student-teacher, fast-inference]
created: 2026-04-21
updated: 2026-04-21
---

A deployment-time stage that produces a faster student model from a trained teacher — reducing denoising steps, eliminating classifier-free-guidance double passes, or both.

**Why it needs to exist as a typed stage** (rather than folding into training). Distillation is a separable concern from base training: the same teacher can spawn multiple students under different compute budgets. The critical non-obvious invariant is that **properties gained at the training stage (e.g. drift robustness from `[[video-world.drift-mitigation-training]]`) do not automatically survive distillation** — if the distillation loss smooths over the training distribution, the student can become brittle at inference even though the teacher was not. Fillers at this stage should declare whether they preserve the teacher's drift properties and how.

**Valid fillers**:
- Single-objective distillation (e.g. DMD, score distillation) retaining the teacher's training-time augmentations so the student inherits robustness.
- Step distillation (many-step → few-step) with or without CFG-distillation.

**Example fillers**:
- `[[dmd-with-self-aug_shen2026]]` — DMD 35-step → 4-step + CFG distilled into the student + **self-augmentation retained during distillation** (Lyra 2.0). ≈13× faster per step.

**Co-requirement**: any filler that claims drift-robustness for its student requires an upstream filler at `[[video-world.drift-mitigation-training]]` — otherwise the "retained" property is vacuous.

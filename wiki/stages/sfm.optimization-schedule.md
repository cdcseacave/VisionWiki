---
title: SfM optimization schedule
type: stage
slug: sfm.optimization-schedule
consumes: [ba-formulation, iteration-budget]
produces: [staged-optimization-plan]
invariants: [warmup-phases-dont-lock-in-bad-minima]
provides_properties: [local-min-escape-routes]
requires_upstream_properties: [ba-formulation-defined]
data_regime: [noisy-init-trajectories]
tags: [sfm, two-stage-ba, robust-loss, cusfm]
created: 2026-04-18
updated: 2026-04-18
---

Which losses run at which stages. Two-stage robust-loss → stringent-outlier schedules (CuSfM) are more robust than single-loss BA when init is noisy. Example fillers: [[cusfm-slam-prior-factor-graph_yu2025]].

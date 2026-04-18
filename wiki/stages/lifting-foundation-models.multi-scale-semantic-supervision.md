---
title: Lifting — multi-scale semantic supervision
type: stage
slug: lifting-foundation-models.multi-scale-semantic-supervision
consumes: [hierarchical-sam-masks, per-mask-clip-embeddings]
produces: [multi-scale-language-field]
invariants: [hierarchy-respects-whole-part-subpart]
provides_properties: [no-query-time-scale-sweep]
requires_upstream_properties: [hierarchical-mask-source]
data_regime: [per-scene]
tags: [langsplat, sam-hierarchy, clip, whole-part-subpart]
created: 2026-04-18
updated: 2026-04-18
---

Bakes multi-scale supervision (whole/part/subpart) into the language field at training — eliminating LERF's query-time multi-scale sweeps. Example fillers: [[sam-mask-hierarchy-supervision_qin2024]].

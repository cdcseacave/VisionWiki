---
title: VLM-reasoning compositional answering
type: stage
slug: vlm-reasoning.compositional-answering
consumes: [evidence-views, query]
produces: [textual-answer]
invariants: [vlm-not-fooled-by-nvs-artifacts]
provides_properties: [compositional-reasoning-output]
requires_upstream_properties: [pretrained-vlm]
data_regime: [inference-time]
tags: [gaussexplorer, vlm]
created: 2026-04-18
updated: 2026-04-18
---

Off-the-shelf VLM consumes evidence views + query → textual answer. Example fillers: [[vlm-view-retrieval-reasoning-3dgs_kim2026]].

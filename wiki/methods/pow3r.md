---
title: Pow3R
type: method
tags: [feed-forward, pointmap, multi-conditioning, vit, croco]
created: 2026-04-15
updated: 2026-04-15
sources: [papers/jang2025_pow3r.md, papers/zhang2025_feed-forward-3d-survey.md]
status: stub
---

## TL;DR
Feed-forward pointmap predictor (DUSt3R lineage) that accepts optional camera intrinsics and depth conditioning at inference, using a CroCo-v2 pretrained ViT backbone. See [Jang et al. 2025](../papers/jang2025_pow3r.md).

## Relation to prior work
- Extends [[dust3r|DUSt3R]] / [[mast3r|MASt3R]] with multi-modal conditioning
- ViT backbone pretrained with [[CroCo]]
- Camera parameterization inspired by [[RayDiffusion]]

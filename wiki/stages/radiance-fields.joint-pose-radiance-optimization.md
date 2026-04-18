---
title: Radiance-fields joint pose-radiance optimization
type: stage
slug: radiance-fields.joint-pose-radiance-optimization
consumes: [initial-cameras-from-sfm, primitive-set, images]
produces: [refined-poses, refined-primitives]
invariants: [ill-conditioned-camera-parameters-preconditioned]
provides_properties: [noise-robust-poses, camera-gradient-through-render]
requires_upstream_properties: [sfm-initial-poses]
data_regime: [noisy-poses, per-scene]
tags: [camp, joint-optimization, pose-refinement]
created: 2026-04-18
updated: 2026-04-18
---

Jointly refines cameras + radiance representation. Requires preconditioning to handle SO(3) + translation + focal's disparate curvatures. Canonical filler is CamP. Representation-agnostic; 3DGS port is Bet #003. Example fillers: [[camera-jacobian-whitening-preconditioner_park2023]].

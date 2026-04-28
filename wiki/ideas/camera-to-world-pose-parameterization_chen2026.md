---
title: Camera-to-world pose parameterization for feed-forward SfM
type: idea
source_paper: wiki/papers/chen2026_lingbot-map.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.camera-parameterization]

inputs: [camera-token-output]
outputs: [camera-to-world-extrinsics-R-t]
assumptions: [pose-head-regresses-extrinsics-as-rotation-plus-translation]
requires_upstream_property: [camera-token-feature]
requires_downstream_property: [trainer-and-evaluator-consume-c2w-format]
learned_params: [pose-head-output-projection]
failure_modes: [weak-evidence-no-isolated-ablation, mechanism-claim-unverified]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [pose-parameterization, camera-output, feed-forward, sfm, unablated-hypothesis]
created: 2026-04-24
updated: 2026-04-24
status: unclaimed
---

## Mechanism

Extrinsic camera pose can be parameterized as either **world-to-camera** ($R_{wc}, t_{wc}$) or **camera-to-world** ($R_{cw}, t_{cw}$), related by $R_{cw} = R_{wc}^T$, $t_{cw} = -R_{wc}^T t_{wc}$. VGGT supervises in world-to-camera; LingBot supervises in camera-to-world.

Paper's claim: in world-to-camera, $t_{wc} = -R_{wc} \cdot \text{camera\_center}$, so small rotation errors in $R_{wc}$ directly rotate the predicted translation. Long sequences accumulate rotation errors, which then amplify translation error — the two are "inherently coupled". Camera-to-world expresses the camera's center $t_{cw}$ directly in world coordinates, which the paper claims decouples translation estimation from rotation estimation.

This is not a standard result — the claim holds that the L2 loss on $t_{cw}$ (where $t_{cw}$ = camera center in world frame) is less sensitive to $R$'s error than the L2 loss on $t_{wc}$. Whether this coupling argument is mechanically precise or heuristic is unclear.

## Why it wins

**⚠ Weak evidence — flagged as unablated hypothesis.**

No isolated ablation in the paper. The claim appears only as justification in §3.3 (Loss Function):

> "Unlike VGGT, we supervise the network using camera-to-world transformations rather than world-to-camera ones. In the world-to-camera parameterization, rotation and translation are inherently coupled, making translation estimation highly sensitive to rotation errors, particularly in long sequences."

No Table-6-style row comparing the two parameterizations. LingBot's overall long-sequence results are strong, but that's attributable to GCA + the other ablated components — not cleanly to this choice.

The argument is plausible but not proven. It could be a post-hoc rationalization for a choice made for other reasons (implementation convenience, compatibility with downstream visualization conventions, etc.).

## Preconditions & compatibility

- Architecture-independent: any feed-forward SfM with a pose regression head can change its output parameterization.
- Requires corresponding updates to the training loss (compare predictions vs GT in the matching format).
- Requires downstream consumers (loop closure, BA backends, visualization) to handle the c2w format.
- Cost: zero additional parameters. Only a representation change.

## Trade-offs vs. alternatives

- **vs. world-to-camera (VGGT baseline)**: mechanism claim favors c2w for long sequences; evidence is weak. Adoption should be considered low-risk low-reward.
- **vs. Plücker ray-bundle parameterization ([[cameras-as-rays|Cameras as Rays]])**: Plücker rays are a more radically different parameterization — no explicit rotation/translation decomposition at all. Orthogonal to the c2w vs. w2c question.
- **vs. quaternion + translation**: standard alternative. Not compared.

## Open questions

- **Is the claimed error-decoupling real?** A focused ablation (w2c vs c2w, same architecture, same training, same data) would settle this. Untested.
- What's the mechanism at the loss-gradient level? If the loss is $\|t_{cw} - t_{cw}^{\text{gt}}\|^2$, the gradient w.r.t. $R$ flows through the chain rule; it's not immediately obvious the decoupling argument holds rigorously.
- Does this interact with the relative pose loss ([[sliding-window-pairwise-pose-loss_chen2026]]) in a surprising way?
- Does the choice matter for *short* sequences, or only long ones (paper's claim)?

## Notes for downstream bets

Any bet that depends on this idea should acknowledge the weak evidence and include a w2c-vs-c2w ablation as part of the validation experiment. Given the bet-filtering rubric (confidence × magnitude ÷ cost), this idea's low confidence penalizes its inclusion in high-stakes bets.

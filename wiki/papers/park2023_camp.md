---
title: "CamP: Camera Preconditioning for Neural Radiance Fields"
type: paper
tags: [nerf, camera-optimization, bundle-adjustment, preconditioning, pose-refinement, zip-nerf]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/park_2023_camp.pdf
url: https://arxiv.org/abs/2308.10902
status: draft
---

📄 [Full paper](../../papers/radiance-fields/park_2023_camp.pdf) · [arXiv](https://arxiv.org/abs/2308.10902)

## TL;DR

CamP proposes a simple yet effective preconditioning technique for joint camera and scene optimization in [[nerf|NeRF]]. By computing a whitening transform from a proxy problem that normalizes and decorrelates camera parameter effects, CamP dramatically improves the conditioning of the joint optimization, reducing rendering error (RMSE) by 67% over non-optimizing baselines and 29% over prior camera-optimizing NeRF methods.

## Problem

[[nerf|NeRF]] requires accurate camera parameters, but [[structure-from-motion]] pipelines like [[colmap|COLMAP]] can produce noisy or incomplete estimates, especially for sparse or wide-baseline captures. Joint optimization of camera parameters alongside the radiance field is an ill-conditioned problem: camera parameters (position, orientation, focal length, distortion) have vastly different scales and are correlated, causing gradient-based optimizers to make slow or unstable progress.

## Method

1. **Proxy problem**: For each camera, the authors compute a Jacobian matrix J of the 2D projections of scene points with respect to camera parameters. They form the scatter matrix S = J^T J.
2. **Whitening transform**: The preconditioner P is derived from S via Cholesky decomposition, yielding a k x k matrix (k = number of camera parameters) that normalizes each parameter's effect and decorrelates them.
3. **Application**: Instead of directly optimizing camera parameters theta, the optimization operates on preconditioned parameters phi, where theta = P * phi + theta_0. This is applied per-camera and is compatible with any camera parameterization (SE3, focal length, distortion coefficients).
4. **Integration**: CamP is applied on top of [[Zip-NeRF]] with minimal implementation effort and negligible computational overhead.

## Results

**Mip-NeRF 360 dataset** (Table 1): FocalPose+Intrinsics with CamP achieves PSNR 28.86, up from 28.27 without any camera optimization and 28.76 without preconditioning.

**Perturbed 360 dataset** (Table 2, cameras deliberately perturbed): SE3+Focal+Intrinsics with CamP achieves PSNR 28.22 / SSIM 0.822 / LPIPS 0.202, vs. 18.50 PSNR with no camera optimization and 27.59 without CamP. Camera orientation error reduced from 0.403 degrees to near-COLMAP accuracy.

**Perturbed synthetic dataset** (Table 3): Consistent improvements across all parameterizations.

**iPhone ARKit captures** (Table 5): CamP enables high-quality NeRFs from casual phone captures without running offline SfM.

## Why it matters

CamP demonstrates that a principled numerical technique (preconditioning) can substantially improve joint camera-scene optimization in neural rendering, without architectural changes or additional training costs. This is broadly applicable to any [[differentiable-rendering]] pipeline and is especially important for making [[nerf|NeRF]] practical with imperfect camera poses from mobile devices or challenging capture conditions.

## Pipeline contribution

- **Per-camera Jacobian-based whitening preconditioner (N1)** — proxy Jacobian $J$ of projection residuals w.r.t. camera params; $P$ = Cholesky of $J^\top J$; optimize $\phi$ where $\theta = P\phi + \theta_0$. candidate thread: [[radiance-field-evolution]] NeRF-family lane · stage: *joint camera + radiance optimization* · replaces/augments: *scheduled frequency (BARF) or raw joint BA* · expected gain: 67% RMSE reduction vs. non-optimizing; 29% over prior camera-optimizing NeRF; recovers from ~0.4° orientation error to near-COLMAP accuracy.
- **Representation-agnostic mechanism (N2)** — preconditioner depends only on the camera-projection Jacobian. candidate thread: [[radiance-field-evolution]] · stage: *joint camera + 3DGS optimization* · synthesis-bet: *port CamP preconditioner to 3DGS pose-refinement residual*. No paper has done this; paper explicitly flags as open.
- **Role**: CamP is the recipe for joint pose+radiance optimization. Its absence from every 3DGS paper is a tell — the 3DGS community uses COLMAP poses and stops there. When a 3DGS paper does try joint optimization (BARF-like), CamP's preconditioning is the standard trick that's missing.

## Relation to prior work

- Builds on [[Zip-NeRF]] (Barron et al., 2023) as the base NeRF model.
- Extends the camera parameterization of SCNeRF (Jeong et al., 2021) with preconditioning.
- Related to [[bundle-adjustment]] in classical [[structure-from-motion]], but applied within the NeRF optimization loop.
- Contrasts with BARF (Lin et al., 2021) which uses coarse-to-fine frequency scheduling instead of preconditioning.
- Compared against Nerfacto, Instant NGP, and various camera parameterizations.

## Open questions / limitations

- The preconditioning does not always prevent convergence to local minima (e.g., white floaters in synthetic scenes caused by misaligned cameras blocking scene content).
- The preconditioner is computed once from initial SfM points; dynamically updating it during training could further improve results.
- Using SfM feature points (rather than uniform samples) to compute the preconditioner may yield better conditioning.
- Applicability to [[3d-gaussian-splatting]] (which also benefits from joint pose optimization) is not explored in this paper.

## References added to the wiki

- [[nerf|NeRF]]
- [[Zip-NeRF]]
- [[colmap|COLMAP]]
- [[structure-from-motion]]
- [[bundle-adjustment]]
- [[differentiable-rendering]]

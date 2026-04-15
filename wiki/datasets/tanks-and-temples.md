---
title: Tanks and Temples
type: dataset
tags: [benchmark, multi-view-stereo, 3d-reconstruction, large-scale, laser-scan, video-input]
created: 2026-04-15
updated: 2026-04-15
sources: [papers/knapitsch2017_tanks-and-temples.md]
code: https://github.com/isl-org/TanksAndTemples
license_code: MIT
license_dataset: CC-BY-4.0
status: stable
---

[code](https://github.com/isl-org/TanksAndTemples)

_Code license: `MIT`_

## What it is

The Tanks and Temples benchmark — 14 real-world scenes captured as 8 MP video with submillimeter laser-scan ground truth and a precision/recall F-score evaluation protocol. Introduced by [Knapitsch et al. 2017](../papers/knapitsch2017_tanks-and-temples.md) (Intel Labs, SIGGRAPH 2017). It is the standard large-scale reconstruction benchmark for [[multi-view-stereo]], [[structure-from-motion]], NeRF, and 3DGS methods.

## Scene splits

| Split | Count | Scenes | GT availability |
|---|---|---|---|
| Training | 7 | Barn, Caterpillar, Church, Courthouse, Ignatius, Meetingroom, Truck | Public |
| Intermediate | 8 | Family, Francis, Horse, Lighthouse, M60, Panther, Playground, Train | Withheld; server evaluation |
| Advanced | 6 | Auditorium, Ballroom, Courtroom, Museum, Palace, Temple | Withheld; server evaluation |

## Evaluation protocol

For each scene, a per-scene distance threshold $\tau$ is computed from nearest-neighbor statistics of the ground-truth point cloud. Given reconstruction $R$ and ground truth $G$:

- **Precision** $P = \frac{|\{r \in R : d(r, G) < \tau\}|}{|R|}$
- **Recall** $R_c = \frac{|\{g \in G : d(g, R) < \tau\}|}{|G|}$
- **F-score** $= \tfrac{2 P R_c}{P + R_c}$

The dual precision+recall framing is the key methodological choice — it separates "accurate where present" from "covers the scene" and prevents papers from gaming a single-number Chamfer metric with thin, sparse reconstructions.

## Why it's still the reference benchmark

- **Submillimeter GT** — laser-scan ground truth outlives any reconstruction method's error floor.
- **Held-out test sets** — the server-based evaluation prevents overfitting to the GT.
- **Video input** — exercises the full pipeline (tracking + dense reconstruction), not a stereo-only subset.
- **Longevity** — nearly every MVS / NeRF / 3DGS / feed-forward reconstruction paper in the wiki reports T&T numbers, making cross-paper comparison possible over a decade.

## Typical use in the wiki

Papers in this wiki that evaluate on T&T (non-exhaustive):
- Gaussian surface extraction: [[papers/radl2026_confidence-mesh-3dgs|CoMe]] (F1 0.521 on T&T), [[papers/kim2025_multiview-geometric-gs|Kim 2025]], [[papers/li2025_va-gs|VA-GS]].
- Outdoor large-scale 3DGS: [[papers/guo2025_ea-3dgs|EA-3DGS]], [[papers/lin2024_vastgaussian|VastGaussian]].
- Radiance fields: [[papers/barron2022_mip-nerf-360|Mip-NeRF 360]], [[papers/barron2023_zip-nerf|Zip-NeRF]], [[papers/sun2025_sparse-voxels-rasterization|SVRaster]].
- Classical MVS baselines: [[papers/schonberger2016_colmap-mvs|COLMAP MVS]].

## Known limitations

- Geometry-only — no material, semantic, or appearance ground truth.
- Static scenes only — not usable for dynamic reconstruction papers.
- Advanced split remains unsaturated; Intermediate split is approaching saturation by recent 3DGS methods.

## Related

- [[papers/knapitsch2017_tanks-and-temples|Knapitsch et al. 2017]] — the benchmark paper
- [[multi-view-stereo]] · [[structure-from-motion]]

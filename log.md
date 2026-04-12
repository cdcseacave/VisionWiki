# Vision Wiki — Activity Log

Append-only, chronological, newest at bottom.
Entry prefixes are grep-friendly: `grep "^## \[" log.md | tail -20`.

## [2026-04-11] bootstrap
- Created: [CLAUDE.md](CLAUDE.md), [index.md](index.md), [log.md](log.md)
- Scaffolded: `raw/papers/`, `raw/assets/`, `wiki/{papers,methods,concepts,datasets,people,threads}/`
- Seeded: [wiki/threads/radiance-field-evolution.md](wiki/threads/radiance-field-evolution.md) as a placeholder for the NeRF → 3DGS lineage
- Notable: wiki begins empty of papers. Domain scope locked to photogrammetry + ML research.

## [2026-04-12] schema-change | Ingest now supports URLs and manages papers/
- Changed: [CLAUDE.md](CLAUDE.md) §1 (directory layout), §1.1 (new: paper storage), §3.1 (ingest workflow), §6.1 (hard rules)
- Added: top-level `papers/` directory with topic subfolders (radiance-fields, feature-matching, sfm-slam, mvs-depth, pose-estimation, mesh-reconstruction, fundamentals, datasets-benchmarks)
- Ingest now accepts: arXiv URLs, direct PDF URLs, project page URLs, or local file paths
- Papers auto-renamed to `<author>_<year>_<short-title>.<ext>` and filed into topic subfolders
- Wiki pages now link to local paper files via `local_paper:` frontmatter and inline links

## [2026-04-12] schema-change | Bare "ingest" = batch mode over raw/
- Changed: [CLAUDE.md](CLAUDE.md) §3.1 — added batch mode as default (no-argument) ingest behavior
- `raw/` now acts as an inbox: drop files there, say "ingest", all unprocessed papers get renamed/moved to `papers/<subfolder>/` and wiki-cascaded
- Originals deleted from `raw/` after confirmed placement in `papers/`
- Already-ingested files are skipped (detected via existing `wiki/papers/` page with matching `local_paper:`)

## [2026-04-12] schema-change | raw/ is pure inbox; added articles/ and assets/ top-level
- Changed: [CLAUDE.md](CLAUDE.md) §1 (layout), added §1.1 (inbox), §1.3 (articles), §1.4 (assets), §1.5 (classification rules)
- `raw/` is now exclusively an inbox — all file types accepted, all moved out on ingest
- `assets/` promoted to top-level (was `raw/assets/`) — permanent store for images/figures
- `articles/` added as top-level permanent store for blog posts, web articles, tutorials
- Classification rules: PDFs→papers/, markdown/HTML→articles/, images→assets/, ambiguous→ask user
- After complete ingest, `raw/` must be empty (this is the "unprocessed work" signal)

## [2026-04-12] seed | Three new research threads
- Created: [wiki/threads/gaussian-to-mesh-pipelines.md](wiki/threads/gaussian-to-mesh-pipelines.md), [wiki/threads/feed-forward-structure-from-motion.md](wiki/threads/feed-forward-structure-from-motion.md), [wiki/threads/mono-depth-estimation.md](wiki/threads/mono-depth-estimation.md)
- Updated: [wiki/threads/radiance-field-evolution.md](wiki/threads/radiance-field-evolution.md) (added cross-links to all three new threads)
- Updated: [index.md](index.md) (four threads now listed)
- Notable: these four threads form a connected graph — radiance fields depend on SfM for poses, mono depth for sparse-view priors, and need mesh extraction for downstream use

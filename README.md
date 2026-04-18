<p align="center">
  <img src="logo.svg" alt="Vision Wiki" width="220">
</p>

# Vision Wiki

A **personal research-synthesis engine** for photogrammetry and machine-learning, maintained collaboratively by a human curator and an LLM that authors and tends the knowledge base.

## Goal

This is not a reading list and not a summarization tool. The goal is to mine every ingested paper for ideas worth recombining, and to evolve a set of **living SOTA pipelines** — one per research thread — that assemble the best current components from across the literature. Each ingest asks two questions that matter: *how does this paper's contribution work at a mechanism level?* and *does it compose with ideas already in our pipelines to produce something better than any single paper has proposed?*

Domains covered:

- Photogrammetry: SfM, MVS, SLAM, bundle adjustment, camera calibration, pose estimation
- Neural scene representations: NeRF, 3D Gaussian Splatting, neural implicit surfaces
- Feature matching, depth estimation, point-cloud and mesh processing
- The ML methods powering them: transformers, diffusion, self-supervised learning

The human drops sources into an inbox and asks questions. The LLM reads deeply, extracts mechanisms, proposes pipeline upgrades and novel cross-paper combinations, flags contradictions, and maintains threads as evolving design documents — never inventing citations.

## How it works

The repo is split into a **source layer** (raw PDFs, articles, assets — git-ignored caches) and a **wiki layer** (committed markdown authored by the LLM). Wiki pages carry a `url:` field so any clone can re-fetch the original paper on demand, making the repo portable without bloating git with binaries.

The wiki's center of gravity is `wiki/threads/`. Each thread page is a **living SOTA pipeline** with explicit stages, a lineage of component swaps, a backlog of candidate components not yet integrated, and an "open questions & synthesis bets" section where the LLM proposes novel combinations of ideas. Paper pages are raw material; threads are where the synthesis happens.

All conventions, templates, and workflows are specified in [CLAUDE.md](CLAUDE.md) — that file is the authoritative schema the LLM follows.

## Directory structure

```
VisionWiki/
├── CLAUDE.md      # schema & operating manual (LLM instructions)
├── README.md      # this file
├── index.md       # human-navigable catalog of every wiki page
├── log.md         # append-only chronological activity log
├── raw/           # INBOX — drop sources here; emptied on ingest
├── papers/        # local PDF cache (git-ignored), re-downloadable via url:
│   ├── radiance-fields/
│   ├── feature-matching/
│   ├── sfm-slam/
│   ├── mvs-depth/
│   ├── pose-estimation/
│   ├── mesh-reconstruction/
│   ├── fundamentals/
│   └── datasets-benchmarks/
├── articles/      # blog posts, tutorials, non-paper text sources
├── assets/        # images, figures, diagrams (git-ignored cache)
└── wiki/          # LLM-authored knowledge base (committed)
    ├── papers/    # one page per ingested paper
    ├── methods/   # algorithms, architectures, techniques
    ├── concepts/  # general ideas and primitives
    ├── datasets/  # benchmarks and training datasets
    ├── people/    # prolific authors and research groups
    ├── threads/   # living SOTA pipelines + novel-combination synthesis bets
    └── designs/   # concrete implementation design docs
```

## Supported functionality

The LLM supports three primary workflows, triggered by keyword.

### 1. `ingest` — bring new sources into the wiki

Four invocation forms:

| Command | Behavior |
|---|---|
| `ingest` (no args) | Batch mode: scan `raw/` for all files, classify each (paper/article/asset), rename per convention, move to the permanent store, and summarize. `raw/` ends empty. |
| `ingest <URL>` | Download the paper (arXiv PDF/HTML, direct PDF, or project page), then process. |
| `ingest <local-path>` | Process an existing local file. |
| `ingest papers/<subfolder>/<file>` | Re-ingest an already-stored paper. |

Each ingest runs Steps 0–8:

0. **Acquire** — download if needed, classify, rename (`<FirstAuthor>_<Year>_<short-title>`), and file into the correct subfolder.
1. **Read** the source fully (no skimming). Auto-downloads missing PDFs when a wiki page has a `url:` but no local file.
2. **Deep analysis — hard gate before any writing.** Produce a structured analysis: goal, foundations, *each* novel contribution with a mechanism-level explanation, the causal story linking each contribution to the headline result, a relation map to existing wiki pages, and pipeline-contribution candidates naming which thread / stage each idea could upgrade. The user reviews before pages are written.
3. **Write the paper page** at `wiki/papers/<key>.md` using the standard template (TL;DR, Problem, Method, Results, Why it matters, **Pipeline contribution**, Relation to prior work, Open questions).
4. **Cascade updates** to referenced concepts, methods, and people — creating stubs only for load-bearing entries.
5. **Evolve affected threads — two passes.** Pass A: per-stage evaluation — does the new component beat the current SOTA pipeline on the thread's success criteria? Swap it in, queue it as a candidate, or flag a contradiction. Pass B (mandatory, even when Pass A changed nothing): holistic synthesis — do ideas compose, do cross-stage interactions invite redesigning multiple stages together, and is there a *novel combination across ≥2 papers* worth proposing as a synthesis bet? Superseded claims are preserved, never silently rewritten.
6. **Update `index.md`** — add new pages, bump `updated:` dates.
7. **Append to `log.md`** with the ingest entry format, including a `Pipeline impact` line and any `Synthesis bet` proposed.
8. **Report back** — list all files created/modified, and state explicitly if Pass B surfaced no new combinations.

A healthy ingest touches 5–15 wiki pages *and* leaves an audit trail in at least one thread's "Current SOTA pipeline", "Candidate components", or "Open questions & synthesis bets" section. An ingest that adds a paper page without touching any thread's pipeline state is a smell: deep analysis or holistic synthesis was skipped.

### 2. `query` — answer questions from the wiki

1. Read `index.md` first to locate relevant pages, then read those pages; only fall back to raw sources when the wiki genuinely doesn't cover the topic.
2. Answer with inline citations (relative markdown links to paper/method pages).
3. If coverage is insufficient, say so explicitly and offer to ingest a source.
4. For substantive synthesis answers, offer to file the result as a new `wiki/threads/<slug>.md`.

### 3. `lint` / `health check` — audit the wiki

Bare `lint` is **read-only** and scans the knowledge base, reporting:

- **Contradictions** between pages
- **Stale claims** superseded by newer sources
- **Orphans** — pages with zero inbound links
- **Missing pages** — concepts referenced ≥3 times with no page
- **Broken wikilinks**
- **Frontmatter drift** — missing `updated:`, empty `sources:`, etc.
- **Thread debt** — threads not updated despite relevant ingests
- **Missing papers** — wiki pages with a `url:` field but no local PDF at `local_paper:`
- **Follow-up suggestions** — 3–5 open questions or source hunts

#### `lint <action>` — targeted fixes

After reviewing the report, the user can invoke a specific fix. Each action targets one diagnostic and always asks for approval before writing.

| Command | What it does | Writes to |
|---|---|---|
| `lint fetch` | Downloads all missing papers (pages with `url:` but no local file). Primary use case: repopulate the `papers/` cache after a fresh `git clone`, since PDFs are git-ignored. | `papers/` only — wiki is not touched |
| `lint frontmatter` | Fills in missing `updated:` dates, empty `sources:`, and other frontmatter drift. | `wiki/` pages |
| `lint orphans` | Proposes inbound wikilinks for orphan pages. | `wiki/` pages |

Contract: **bare `lint` is always safe and read-only; `lint <action>` may write but always asks first.**

## Page conventions

Every wiki page carries YAML frontmatter:

```yaml
---
title: <canonical title>
type: paper | method | concept | dataset | person | thread | design
tags: [nerf, differentiable-rendering, ...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [papers/kerbl2023_3dgs.md, ...]
local_paper: papers/<subfolder>/<filename>.pdf   # paper pages only
url: https://arxiv.org/abs/XXXX.XXXXX            # paper pages only
status: stub | draft | stable | contested
---
```

- **Filenames**: lowercase-kebab-case. Papers use `<firstauthor><year>_<slug>.md`.
- **Cross-links**: Obsidian wikilinks `[[3d-gaussian-splatting]]` for concepts/methods/people/threads; relative markdown links for paper citations.
- **Math**: MathJax-compatible — inline `$...$`, display `$$...$$`.
- **Figures**: reference local assets; never hot-link external URLs.
- **Citations**: every factual claim must trace to a `sources:` entry, or be flagged `> [!needs-source]`.

Templates for paper, method, and thread pages are documented in [CLAUDE.md §2](CLAUDE.md).

## Hard rules

1. Source papers in `papers/` and `raw/` are **read-only** after placement.
2. **Never invent citations** — flag missing sources explicitly.
3. **Never silently overwrite** a contradicted claim — preserve it with a "superseded by" note.
4. **Stay in scope**: photogrammetry and ML research only.
5. **Don't over-create pages** — a name-drop is not a page.
6. **Cite down to the page** — `sources:` must back every factual claim.
7. **Obsidian is the read-side UI** — wikilinks and frontmatter should work natively.
8. **No emojis in wiki content** unless requested.

## Usage

1. Drop papers, articles, or images into `raw/` with any filename.
2. Ask the LLM to `ingest` — it classifies, files, reads, summarizes, and cross-links everything.
3. Ask questions — the LLM answers from the wiki with citations.
4. Periodically run `lint` to surface contradictions, stale claims, and gaps.

The authoritative operating manual is [CLAUDE.md](CLAUDE.md). The schema co-evolves with usage; changes are logged under `## [YYYY-MM-DD] schema-change` entries in [log.md](log.md).

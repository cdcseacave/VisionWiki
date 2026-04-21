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

Four page types carry the synthesis:

- **Paper pages** (`wiki/papers/`) — one per ingested source; records what the paper claims and points at the ideas it contributed.
- **Idea pages** (`wiki/ideas/`) — **the atomic unit of synthesis.** Every distinct novel contribution becomes a first-class page with a mechanism description, typed I/O contracts, assumptions, and composition edges (`requires:`, `unlocks:`, `co_requires:`, `refines:`, `equivalent_to:`, `contradicts:`). Questions like "does X compose with Y?" become graph queries instead of re-derivation from prose.
- **Stage pages** (`wiki/stages/`) — typed slots in a pipeline (e.g. `radiance-fields.rendering`, `sfm.initial-pair-selection`) with `consumes:` / `produces:` types and nonlocal `provides_properties:`. Thread pipelines are DAGs of stages, filled by ideas whose `stages:` match — making composition type-checkable.
- **Thread pages** (`wiki/threads/`) — living per-goal pipelines, each with up to 3 **operating points** (e.g. `op:realtime` vs. `op:offline-quality`) so one thread can hold multiple regime-specific SOTA pipelines side-by-side. Threads track a SOTA pipeline DAG, a lineage of component swaps, candidate components not yet integrated, structured synthesis bets (novel cross-paper combinations with a rubric of magnitude × confidence ÷ cost ÷ breakage), and a capability-gaps shopping list for future ingests.

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
    ├── ideas/     # atomic novel contributions extracted from papers
    ├── stages/    # typed pipeline stages (input/output/invariant schemas)
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

0. **Acquire** — download if needed, classify, rename (`<FirstAuthor>_<Year>_<short-title>`), file into the correct subfolder, then **find official/canonical code** (paper body → project page → Papers-with-Code → GitHub) and **identify licenses** (`license_paper:`, `license_code:`, `license_dataset:` — SPDX when possible, verbatim + gloss otherwise). Non-commercial / research-only / unknown licenses are flagged in the paper page body.
1. **Read** the source fully (no skimming). Auto-downloads missing PDFs when a wiki page has a `url:` but no local file.
2. **Deep analysis — hard gate before any writing.** Produce a structured analysis: goal, foundations, and — for *each* novel contribution — an **idea candidate** with proposed slug, mechanism-level explanation, draft structured fields (`scope`, `stages`, `inputs`, `outputs`, `assumptions`, `learned_params`, `failure_modes`, `co_requires`), a **scope classification** (`drop-in` / `stage-swap` / `multi-stage-collapse` / `stage-split` / `topology-rewrite` / `new-paradigm` / `bridge` — smallest honest scope), **bundle detection** (`co_requires:` siblings), **bridge detection** (I/O mismatch with neighbors), and an **equivalence check** against existing ideas. Finally, for each idea, a pipeline-contribution candidate naming which thread / stage / **operating point** could absorb it. The user reviews before pages are written.
3. **Write paper page + idea pages + any new stage pages.** `wiki/papers/<key>.md` records the source; each enumerated novel contribution becomes a `wiki/ideas/<slug>.md` with full composition frontmatter (or extends `also_in:` / `refines:` on an existing idea). Unbound stage slugs get a `wiki/stages/<slug>.md` stub. The paper page's "Pipeline contribution" section lists wikilinks to the created ideas — mechanism lives on the idea page, not re-stated on the paper page. **No back-fill escape hatch**: every novel contribution from Step 2 must produce or update an idea page here.
4. **Cascade updates** to referenced concepts, methods, and people — creating stubs only for load-bearing entries.
5. **Evolve affected threads — two passes.** Pass A: **per-OP, per-stage evaluation** — type-check the idea's I/O against stage `consumes:` / `produces:`, check upstream `provides_properties:` satisfy the idea's `requires_upstream_property:`, check assumptions against the **assumption conflict registry** (§7 of CLAUDE.md), verify bundles (`co_requires:` siblings all adoptable), apply the scope-appropriate DAG change (drop-in / collapse / split / rewrite / new-paradigm / bridge), then decide per operating point: does it beat the OP's SOTA? Swap it in, queue as candidate, or flag contradiction. An idea may win on one OP and lose on others. Pass B (mandatory, even when Pass A changed nothing): holistic synthesis — traverse `requires:` / `unlocks:` / `refines:` edges against every existing idea (including orphan ideas), check cross-stage assumption shifts, propose ≥1 **novel cross-paper combination** as a structured synthesis bet with the `magnitude × confidence ÷ cost ÷ breakage_risk` rubric, consider cross-thread transfer, and **refresh Capability gaps**. Superseded claims are preserved, never silently rewritten.
6. **Update `index.md`** — add new pages, bump `updated:` dates.
7. **Append to `log.md`** with the ingest entry format, including a `Pipeline impact` line (per affected `thread:op`) and any `Synthesis bet` proposed.
8. **Report back** — list all files created/modified, and state explicitly if Pass B surfaced no new combinations.

A healthy ingest touches 5–15 wiki pages *and* leaves an audit trail in at least one thread's SOTA pipeline, Candidate components, Open questions & synthesis bets, or Capability gaps. An ingest that adds a paper page without touching any thread's pipeline state is a smell: deep analysis or holistic synthesis was skipped.

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
- **Stale threads** — threads whose `updated:` predates any of their cited sources
- **Missing papers** — wiki pages with a `url:` field but no local PDF
- **Missing code / licenses** — papers, methods, datasets with unfilled `code:` / `license_*:` fields (or a `no code found` marker older than 6 months)
- **Restrictive licenses** (informational) — non-commercial / research-only / unknown
- **Follow-up suggestions** — 3–5 open questions or source hunts

#### `lint <action>` — targeted fixes

After reviewing the report, the user can invoke a specific fix. Each action targets one diagnostic and always asks for approval before writing.

| Command | What it does | Writes to |
|---|---|---|
| `lint fetch` | Downloads all missing papers (pages with `url:` but no local file). Primary use case: repopulate the `papers/` cache after a fresh `git clone`. | `papers/` cache only |
| `lint frontmatter` | Fills in missing `updated:` dates, empty `sources:`, and other frontmatter drift. | `wiki/` pages |
| `lint orphans` | Proposes inbound wikilinks for orphan pages. | `wiki/` pages |
| `lint stale-threads` | Threads behind their own cited sources → targeted per-thread re-cascade. | `wiki/threads/*.md` |
| `lint find-code` | Re-searches for official/canonical code on pages lacking it (or with stale `no code found` markers). Fills `code:` + `license_code:`. | `wiki/papers,methods,datasets/*.md` |
| `lint licenses` | Fills missing `license_paper:` / `license_code:` / `license_dataset:` on pages whose underlying resource exists. | `wiki/papers,methods,datasets/*.md` |
| `lint idea-duplicates` | Surfaces candidate duplicate ideas (same stage + overlapping mechanism) for manual merge via `equivalent_to:` / `refines:`. | `wiki/ideas/*.md` |
| `lint unlocks-fired` | New ideas whose slugs match a shelved bet's `triggers:` → surface for re-evaluation. | read-only |
| `lint stage-coverage` | Stages referenced with no page, or stage pages with no fillers; idea × stage × thread coverage matrix. | read-only |
| `lint synthesis-pressure` | Per thread: ingests vs. new Pass B bets over trailing N days. Low ratio = regressing to reading-list mode. | read-only |
| `lint type-check` | Per-node I/O compatibility, `co_requires:` bundle presence, and **assumption-vector compatibility** against the §7 conflict registry. | read-only |
| `lint topology-drift` | Thread SOTA nodes/edges vs. cumulative Pipeline lineage — detects nodes without lineage entries, or lineage referencing retired nodes. | read-only |
| `lint bridge-coverage` | Thread nodes with incompatible-I/O neighbors and no bridge → candidate bridge-idea stubs. | `wiki/ideas/*.md` stubs |
| `lint bets` | Sorts open bets by `magnitude × confidence ÷ cost ÷ breakage_risk`; flags missing rubric fields and bets stale >30 days. | read-only |
| `lint design-closure` | Three-way reconciliation: designs whose `outcome:` is non-pending but the bet is still `in-design`; validated bets whose ideas lack `validated_in:`; orphan designs. | read-only |
| `lint orphan-ideas` | Ideas referenced by 0 bets older than 30 days. | read-only |

Contract: **bare `lint` is always safe and read-only; `lint <action>` may write but always asks first.**

## Page conventions

Every wiki page carries YAML frontmatter:

```yaml
---
title: <canonical title>
type: paper | method | concept | dataset | person | thread | design | idea | stage
tags: [nerf, differentiable-rendering, ...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [papers/kerbl2023_3dgs.md, ...]
local_paper: papers/<subfolder>/<filename>.pdf   # paper pages only
url: https://arxiv.org/abs/XXXX.XXXXX            # paper pages only
code: https://github.com/<org>/<repo>            # paper / method / dataset; omit if none found
license_paper: CC-BY-4.0                         # paper pages only
license_code: MIT                                # paper / method / dataset
license_dataset: CC-BY-NC-4.0                    # dataset pages only
status: stub | draft | stable | contested
---
```

Type-specific extensions (defined in [CLAUDE.md §2.1](CLAUDE.md)):

- **Thread pages** carry `operating_points: [op:default, op:realtime, ...]` (≤3).
- **Idea pages** carry `scope:`, `stages:`, `inputs:`, `outputs:`, `assumptions:`, `learned_params:`, `failure_modes:`, plus the composition graph (`requires:`, `unlocks:`, `co_requires:`, `bridges:`, `equivalent_to:`, `refines:`, `contradicts:`) and `validated_in:`.
- **Stage pages** carry `slug: <domain>.<slot>`, `consumes:`, `produces:`, `invariants:`, `provides_properties:`, `requires_upstream_properties:`, `data_regime:`.
- **Design pages** carry `realizes_bet:`, `realizes_ideas:`, `outcome:`.

Conventions:

- **Filenames**: lowercase-kebab-case. Papers use `<firstauthor><year>_<slug>.md`; ideas use `<descriptor>_<firstauthor><year>.md`; stages use their dotted `<domain>.<slot>` slug.
- **Cross-links**: Obsidian wikilinks `[[3d-gaussian-splatting]]` for concepts/methods/people/threads/ideas/stages; relative markdown links for paper citations.
- **Math**: MathJax-compatible — inline `$...$`, display `$$...$$`.
- **Figures**: reference local assets; never hot-link external URLs.
- **Citations**: every factual claim must trace to a `sources:` entry, or be flagged `> [!needs-source]`.

Full templates for every page type are documented in [CLAUDE.md §2](CLAUDE.md).

## Hard rules

1. Source papers in `papers/` and `raw/` are **read-only** after placement.
2. **Never invent citations** — flag missing sources explicitly.
3. **Never silently overwrite** a contradicted claim — preserve it with a "superseded by" note.
4. **Stay in scope**: photogrammetry and ML research only.
5. **Don't over-create pages** — a name-drop is not a page.
6. **Cite down to the page** — `sources:` must back every factual claim.
7. **Obsidian is the read-side UI** — wikilinks and frontmatter should work natively.
8. **No emojis in wiki content** unless requested.
9. **Pareto cap**: a thread has ≤3 operating points. Need a 4th → split the thread.
10. **Goal first**: every thread has a `## Goal`; `## Goal contract` is optional but drafted at creation and user-approved before commit.
11. **Ideas are first-class**: every novel contribution lives in `wiki/ideas/<slug>.md`; threads and bets reference by wikilink. No back-fill escape hatch on ingest.
12. **Type-check compositions**: filler idea's I/O must be compatible with the node's stage; upstream `provides_properties:` must satisfy the filler's `requires_upstream_property:`. Mismatches → retype or add a bridge.
13. **Topology honesty**: `multi-stage-collapse` / `stage-split` / `topology-rewrite` / `new-paradigm` ideas are not 1:1 stage fillers; the thread DAG reflects the actual change and records a topology-change lineage entry.
14. **Bundles travel together**: `co_requires:` ideas must all be present as fillers in the same pipeline — partial adoption is a type-check failure.
15. **Licenses never block bets or SOTA choices**: license info is informational; bet adoption and SOTA composition are decided on mechanism merit alone.

An **assumption conflict registry** ([CLAUDE.md §7](CLAUDE.md)) lists pairs of mutually exclusive assumptions (e.g. `static-scene` vs. `dynamic-objects`, `posed-input` vs. `unposed-input`) that `lint type-check` uses to catch pipelines whose adopted ideas have silently incompatible regimes.

## Usage

1. Drop papers, articles, or images into `raw/` with any filename.
2. Ask the LLM to `ingest` — it classifies, files, reads, summarizes, and cross-links everything.
3. Ask questions — the LLM answers from the wiki with citations.
4. Periodically run `lint` to surface contradictions, stale claims, and gaps.

The authoritative operating manual is [CLAUDE.md](CLAUDE.md). The schema co-evolves with usage; changes are logged under `## [YYYY-MM-DD] schema-change` entries in [log.md](log.md).

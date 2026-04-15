# Vision Wiki — Schema & Operating Manual

This is a **personal research-synthesis engine** for photogrammetry and
machine-learning. It is not a reading list and not a summarization tool. Its
end-product is **novel pipelines**: each thread maintains a current
best-known end-to-end pipeline for a problem, and every ingested paper is
mined for ideas that could improve that pipeline — either by replacing a
stage, or by unlocking a *new combination* of ideas that no single paper has
tried. Deep mechanism-level understanding of each paper, and holistic
reasoning about how ideas compose across papers, are the two activities the
wiki exists to support.

You (the LLM) are the sole author and maintainer of the `wiki/` layer. The
human curates sources and asks questions. Never invent citations. A shallow
summary is worse than useless — if you can't explain the mechanism of a new
idea, you don't yet understand the paper.

Domain focus: photogrammetry, SfM, MVS, SLAM, NeRF / 3D Gaussian Splatting,
neural implicit surfaces, feature matching, bundle adjustment, camera calibration,
point-cloud / mesh processing, depth estimation, pose estimation, and the ML
methods that power them (transformers, diffusion, self-supervised learning, etc.).

---

## 1. Directory layout

```
VisionWiki/
├── CLAUDE.md              # this file — the schema
├── index.md               # content-oriented catalog of every wiki page
├── log.md                 # chronological append-only activity log
├── raw/                   # INBOX — drop anything here, emptied on ingest (see §1.1)
├── papers/                # LOCAL CACHE (git-ignored): PDFs, re-downloadable via url: (see §1.2)
│   ├── radiance-fields/   # example subfolder — see §1.2 for taxonomy
│   ├── feature-matching/
│   └── ...
├── articles/              # permanent store: blog posts, web articles, notes (see §1.3)
├── assets/                # LOCAL CACHE (git-ignored): images, figures, diagrams (see §1.4)
└── wiki/                  # LLM-authored, constantly-maintained knowledge base
    ├── papers/            # one page per ingested paper (the "source summary")
    ├── methods/           # algorithms, architectures, techniques (e.g. `3d-gaussian-splatting.md`)
    ├── concepts/          # general ideas and primitives (e.g. `bundle-adjustment.md`)
    ├── datasets/          # benchmark + training datasets
    ├── people/            # prolific authors / research groups
    ├── threads/           # evolving syntheses, open questions, comparisons
    └── designs/           # implementation design documents (concrete "how to build it" plans)
```

### 1.1 Inbox (`raw/`)

`raw/` is a **drop zone / inbox**. The user dumps any files here — PDFs, markdown
clippings, images, blog post exports, screenshots — with whatever messy names
they have. The LLM never reads directly from `raw/` during queries.

On ingest (bare `ingest` with no arguments), the LLM:
1. Scans `raw/` recursively for all files.
2. Classifies each file by type (paper, article, or asset).
3. Renames and moves it to the appropriate permanent store (`papers/`, `articles/`, or `assets/`).
4. Deletes the original from `raw/`.

**After a complete ingest, `raw/` should be empty.** A non-empty `raw/` means
there are unprocessed sources waiting. This is the primary signal to the user
(and the LLM) that work remains.

### 1.2 Paper storage (`papers/`)

The top-level `papers/` directory is a **local cache** of source papers (PDFs,
markdown clippings, arXiv HTML exports). It is **not committed to git** (listed
in `.gitignore`) because PDFs are large and binary. The wiki pages in
`wiki/papers/` *are* committed — they contain the `url:` field that allows
any clone to re-download the original paper on demand.

The LLM manages this folder — it downloads, renames, and organizes papers here.

**Cache behavior**: when reading a paper (for ingest or query), if the local
file at `local_paper:` is missing but `url:` exists in the wiki page's
frontmatter, download the paper from the URL first:
- arXiv URLs: `curl -L -o <local_paper_path> https://arxiv.org/pdf/<id>`
- Other URLs: `curl -L -o <local_paper_path> <url>`
Then proceed normally. This makes the wiki portable — clone the repo, and
papers are fetched on first access.

**Naming convention**: `<FirstAuthor>_<Year>_<Short-Title>.<ext>`
- lowercase, kebab-case for the short title
- Examples: `kerbl_2023_3d-gaussian-splatting.pdf`,
  `mildenhall_2020_nerf.pdf`, `wang_2024_dust3r.md`

**Subfolder taxonomy** — organize papers into topic subfolders. Create new
subfolders as needed; merge or restructure when a subfolder grows past ~20
files. Current seed taxonomy:

| Subfolder | Scope |
|-----------|-------|
| `radiance-fields/` | NeRF, 3DGS, neural implicit surfaces, novel-view synthesis |
| `feature-matching/` | keypoint detection, descriptor learning, matching pipelines |
| `sfm-slam/` | structure from motion, visual SLAM, visual odometry |
| `mvs-depth/` | multi-view stereo, monocular/stereo depth estimation |
| `pose-estimation/` | camera pose, object pose, PnP, relative/absolute pose regression |
| `mesh-reconstruction/` | surface reconstruction, meshing, point-cloud processing |
| `fundamentals/` | general ML methods, transformers, diffusion, optimization |
| `datasets-benchmarks/` | dataset papers, benchmark comparisons |

If a paper spans multiple categories, file it under its **primary contribution**
and note the cross-topic relevance in the wiki page.

### 1.3 Article storage (`articles/`)

Blog posts, web articles, tutorial pages, and non-paper text sources.
Renamed on ingest to `<source>_<year>_<short-title>.md` (e.g.
`matthewtancik_2023_nerfstudio-tutorial.md`). Organized in the same topic
subfolders as `papers/` where applicable, or a flat structure if volume is low.

### 1.4 Asset storage (`assets/`)

Images, figures, diagrams, and screenshots extracted from or related to
sources. Renamed on ingest to `<paper-or-article-key>_<descriptor>.<ext>`
(e.g. `kerbl_2023_3dgs_pipeline-overview.png`). Wiki pages reference them as:

```markdown
![3DGS pipeline](../assets/kerbl_2023_3dgs_pipeline-overview.png)
```

### 1.5 Classification rules for ingest

| File type | Destination |
|-----------|-------------|
| PDF (research paper, arXiv, conference) | `papers/<subfolder>/` |
| Markdown / HTML (blog post, tutorial, web clip) | `articles/<subfolder>/` or `articles/` |
| Image (PNG, JPG, SVG, diagram) | `assets/` |
| PDF (non-paper: slide deck, report) | `articles/` |
| Unknown / ambiguous | Ask the user before filing |

## 2. Page conventions

Every wiki page is a markdown file with YAML frontmatter:

```yaml
---
title: <canonical title>
type: paper | method | concept | dataset | person | thread | design
tags: [nerf, differentiable-rendering, ...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [papers/kerbl2023_3dgs.md, papers/mildenhall2020_nerf.md]
local_paper: papers/<subfolder>/<filename>.pdf   # paper pages only
url: https://arxiv.org/abs/XXXX.XXXXX            # paper pages only — external URL
code: https://github.com/<org>/<repo>            # paper / method / dataset pages — official or canonical implementation; omit if none found at ingest time
license_paper: CC-BY-4.0                         # paper pages only — paper license (arXiv / conference page); "unknown" if it cannot be determined
license_code: MIT                                # paper / method / dataset pages — license of the repository at `code:`; omit if `code:` is omitted, "unknown" if repo exists but no LICENSE file
license_dataset: CC-BY-NC-4.0                    # dataset pages only — data license; "unknown" if not stated
status: stub | draft | stable | contested
---
```

**Code and license fields**:
- `code:` must be an *official* or *canonical community* implementation (authors' own repo, or the most-used community port when authors release none). Do **not** link random forks. If no code can be found after a reasonable search (Google + Papers-with-Code + GitHub search for first-author + title), omit the field — `lint find-code` exists to re-check over time.
- `license_paper:` usually found at the arXiv abstract page footer (arXiv submission license), on the publisher page (CC-BY / ACM DL / Springer), or in the PDF's first page. Record as SPDX identifier when possible (`CC-BY-4.0`, `CC-BY-NC-4.0`, `arxiv-nonexclusive`, `ACM`, `IEEE`, `Springer`, `unknown`).
- `license_code:` read from the repo's `LICENSE` / `COPYING` file or the GitHub sidebar badge. SPDX form (`MIT`, `Apache-2.0`, `BSD-3-Clause`, `GPL-3.0`, `AGPL-3.0`, `non-commercial`, `research-only`, `unknown`). **Flag every non-commercial / research-only license explicitly** — these materially affect whether a pipeline component is usable downstream.
- `license_dataset:` same SPDX discipline; common values `CC-BY-4.0`, `CC-BY-NC-4.0`, `custom-research`, `unknown`.

**`sources:` semantics by page type**:
- **method / concept / thread / dataset / person pages**: `sources:` lists the wiki paper pages that back the claims on this page. Required, non-empty.
- **paper pages**: a paper page *is* its own primary source. Leave `sources:` omitted, or use it to list the **prior work this paper builds on** (other paper pages in the wiki). Either is acceptable; an empty `sources: []` on a paper page is not a lint violation.
- **design pages**: `sources:` lists the wiki pages (any type) that the design rests on.

- **Filenames**: lowercase-kebab-case. For papers use `<firstauthor><year>_<slug>.md`
  (e.g. `kerbl2023_3dgs.md`). For methods/concepts use the canonical name
  (e.g. `bundle-adjustment.md`, `3d-gaussian-splatting.md`).
- **Cross-links**: use Obsidian wikilinks `[[3d-gaussian-splatting]]` for
  concept/method/person/thread links. Use relative markdown links for paper
  citations, e.g. `[Kerbl et al. 2023](papers/kerbl2023_3dgs.md)`.
- **Math**: inline `$...$`, display `$$...$$` (MathJax-compatible).
- **Figures**: reference locally stored images as
  `![caption](../assets/<file>)`. Never hot-link external URLs.
- **Citations**: every factual claim on a wiki page must trace back to a
  specific entry in `sources:` frontmatter. If you can't cite it, flag it with
  `> [!needs-source]`.

### Page templates

**Paper page** (`wiki/papers/<key>.md`):
```markdown
📄 [Full paper](../../papers/<subfolder>/<filename>.pdf) · [arXiv](https://arxiv.org/abs/XXXX.XXXXX) · [code](https://github.com/<org>/<repo>)

_Paper license: `<spdx>` · Code license: `<spdx>`_   <!-- omit code half if no code; write "no code found" if searched and none located -->


## TL;DR
One paragraph. What is the contribution in one breath?

## Problem
What limitation of prior work does this address?

## Method
Core technical idea. Math where it clarifies. Pseudocode if useful.

## Results
Headline numbers, datasets, comparisons. Only cite numbers that appear in the paper.

## Why it matters
How this fits into the broader [[thread-name]] and what it enables downstream.

## Pipeline contribution
For each novel idea from this paper, state how it could plug into a thread's
SOTA pipeline. One bullet per idea:

- **<idea name>** — mechanism (1–2 sentences) · candidate thread: [[thread-slug]] · stage: `<pipeline stage>` · replaces/augments: `<current component>` · expected gain: `<metric / ablation that supports this>` · risks / assumptions: `<what must hold>`

If no thread currently owns the relevant stage, propose whether a new thread
should exist rather than filing it nowhere.

## Relation to prior work
- Builds on [[method]] / [Author Year](papers/...)
- Contrasts with [[method]]

## Open questions / limitations
Bulleted. Both the paper's stated limitations and your own skepticism.

## References added to the wiki
Pages created or meaningfully updated by this ingest.
```

**Method page** (`wiki/methods/<name>.md`):
```markdown
## What it is
One-paragraph definition.

## How it works
Key mechanics. Math, diagrams, pseudocode.

## Variants & lineage
Chronological: origin → notable successors. Link to paper pages.

## Strengths
## Limitations
## Typical use in photogrammetry/ML pipelines
## Key references
```

**Thread page** (`wiki/threads/<name>.md`): a *living SOTA pipeline*, not just
an annotated reading list. A thread exists to answer: "given everything we've
ingested, what is the current best end-to-end pipeline for this problem, and
what novel combinations of ideas could beat it?" Example slugs:
`radiance-field-evolution.md`, `feature-matching-learned-vs-classical.md`.

Required sections:

```markdown
## Goal & success criteria
What the pipeline is trying to do. What "better" means — metric(s), dataset(s),
or qualitative target. This is what every component swap is judged against.

## Current SOTA pipeline (as of YYYY-MM-DD)
Stage-by-stage — numbered list or diagram. For each stage name:
- the current best component,
- the paper it comes from (linked),
- the measured / claimed gain over the prior choice,
- known failure modes or assumptions it imposes on neighbouring stages.

## Pipeline lineage
Chronological record of component swaps and pipeline-level redesigns.
Format: `YYYY-MM-DD · <stage>: <old> → <new>` · driver: [paper link] · rationale.
Prior pipelines are preserved (strikethrough or collapsed) — never silently
rewritten (Hard Rule §6.3).

## Candidate components / not yet integrated
Ideas from ingested papers that look promising but haven't been promoted into
the pipeline. Each entry: component · source paper · which stage it targets ·
why it's not yet in (needs ablation, incompatible assumptions, untested
combination, waiting on a prerequisite idea, etc.).

## Open questions & synthesis bets
Where the LLM proposes *novel combinations* not tried in any single paper.
Each bet: hypothesis · ideas being combined (≥2 papers) · expected gain · risk ·
what experiment would validate it. This section is the project's research
agenda — keep it alive.

## Contradictions & tensions
Places where ingested papers disagree. The thread's current resolution and
confidence level. Link the conflicting sources.

## Sources
(populated from frontmatter `sources:`)
```

Reread and revise on every relevant ingest. The "Current SOTA pipeline" and
"Open questions & synthesis bets" sections together are the thread's reason
to exist — if an ingest touches a thread without updating at least one of
them (or explicitly noting "no change warranted"), the synthesis step was
skipped.

## 3. Workflows

### 3.1 Ingest

The user can trigger an ingest in four ways:

| Invocation | Behavior |
|------------|----------|
| **`ingest`** (no arguments) | **Batch mode.** Scan `raw/` for all unprocessed files (PDFs, markdown, HTML). Process each one through Steps 0–8. This is the default workflow — drop files into `raw/`, then say "ingest". |
| **`ingest <URL>`** | Download the paper, then process through Steps 0–8. |
| **`ingest <local-path>`** | Process an existing local file through Steps 0–8. |
| **`ingest papers/<subfolder>/<file>`** | Re-ingest an already-stored paper (skip Step 0 download/rename/move). |

#### Step 0 — Acquire the paper(s)

**Batch mode** (no arguments):
1. List all files in `raw/` recursively.
2. Classify each file per §1.5 (paper → `papers/`, article → `articles/`,
   image → `assets/`, ambiguous → ask user).
3. Present the user with a numbered list before proceeding:
   _"Found N files in raw/. Plan: 1. <filename> → papers/radiance-fields/
   2. <filename> → assets/ …"_
4. On approval, process each: rename, move to permanent store, run
   single-paper acquisition below for papers/articles. Images just get
   renamed and moved to `assets/`.
5. After all files are processed, `raw/` should be empty.

**Single-paper acquisition** (URL or local file):

| Input | Action |
|-------|--------|
| **arXiv URL** (abs or pdf) | Download the PDF via `curl -L` to a temp location. If an arXiv HTML version is available (`/html/<id>`), prefer downloading that as markdown (better for reading). |
| **Direct PDF URL** | Download via `curl -L`. |
| **Project page / blog URL** | Use WebFetch or curl to grab the page content. If a PDF link is found, download that too. |
| **Local file in `raw/`** | Process in place (will be moved out of `raw/` after). |
| **Local file elsewhere** | No download needed. |

After acquiring:
1. **Determine the canonical title, first author, and year** from the paper
   content or arXiv metadata.
2. **Classify** the file per §1.5 (paper, article, or asset).
3. **Rename** per the target store's convention (§1.2, §1.3, or §1.4).
4. **Move** the file into the appropriate permanent store (`papers/<subfolder>/`,
   `articles/`, or `assets/`). Create subfolders as needed.
5. **If the source came from `raw/`**: the move itself empties it from the
   inbox. Verify `raw/` is clean after batch ingest.
6. **Find the code** (papers / datasets only). Search in this order:
   a. the paper's abstract / conclusion / footnotes / GitHub badge;
   b. the project page (if the URL is a project page, not arXiv);
   c. Papers-with-Code entry for the title;
   d. GitHub search for `<first-author> <short-title>` and `<short-title>` in the paper's topic area.
   Accept only the **official** repo (authors' own) or, if authors released none, the **canonical** community implementation most cited / forked in follow-up work. Record as `code:` in frontmatter. If the search returns nothing, omit the field — do **not** guess; `lint find-code` will re-check later.
7. **Identify the licenses**:
   - `license_paper:` — read from arXiv (submission license at the abstract-page footer), or the publisher page / PDF first page for conference versions. Record SPDX-style.
   - `license_code:` — if `code:` is set, read the repo's `LICENSE` file or the GitHub sidebar license badge. If a repo exists but has no detectable license, record `unknown` (this is an actual signal — unlicensed code is effectively all-rights-reserved and needs flagging).
   - `license_dataset:` — for dataset pages only; read the dataset's release page / README / `LICENSE`.
   Flag non-commercial, research-only, and `unknown` license-code results explicitly in the paper-page body under *"Code & license"* (see template) — these materially affect downstream usability and must not be buried in frontmatter.
8. Report the final path to the user (e.g. "Saved to
   `papers/radiance-fields/kerbl_2023_3d-gaussian-splatting.pdf` · code: `github.com/graphdeco-inria/gaussian-splatting` · paper license: `arxiv-nonexclusive` · code license: `non-commercial`").

#### Step 1 — Read the source

**Cache check**: if the local file at `local_paper:` does not exist but the
wiki page has a `url:` field, download the paper first:
- arXiv: `curl -L -o <local_paper_path> https://arxiv.org/pdf/<id>`
- Other: `curl -L -o <local_paper_path> <url>`
Create any missing subdirectories under `papers/` before downloading.

Read the source fully. For PDFs, read all pages. For arXiv markdown,
read the whole file. Don't skim.

#### Step 2 — Deep analysis (hard gate before any writing)

Understanding a paper is the whole point of this project. A shallow summary is
worse than useless — it pollutes the wiki and blocks synthesis. Before any
file is written, produce a structured analysis and present it to the user for
review. This is the hard gate between reading and writing.

Required analysis artifacts:

1. **Goal.** What problem does the paper attack, in the paper's own framing?
   What does "solved" look like for them?
2. **Foundations.** Which prior methods / concepts / datasets does it rest on?
   Link every foundation to an existing wiki page. Flag gaps where a load-bearing
   foundation has no page yet (candidate for a Step 4 stub).
3. **Novel contributions — enumerated.** List each *distinct* new idea /
   feature / method / architectural choice separately. For each:
   - **What it is** (one line).
   - **How it works at mechanism level** — not "they use contrastive loss" but
     *what* is contrasted against *what*, under *which* invariance, driven by
     *which* signal. Math or pseudocode where it clarifies. If you can't
     explain the mechanism, you don't yet understand the paper — read again.
4. **Why it wins.** For each novel contribution, give the causal story linking
   it to the headline result: which ablation / table / comparison in the paper
   actually supports the claim, and what the measured effect is. If no
   ablation isolates the contribution, say so — that's a weakness worth
   flagging.
5. **Relation map.** For each contribution, how does it relate to existing
   wiki pages? Extends / replaces / contradicts / is orthogonal to. Cite the
   wiki pages.
6. **Pipeline contribution candidates.** For each novel contribution, which
   existing thread's SOTA pipeline could absorb it, at which stage, and what
   it would replace or augment? If no thread owns the stage, propose whether
   a new thread should exist. This feeds directly into Step 5.

Present the analysis to the user. Ask whether anything should be emphasized,
reframed, or deepened before pages are written. Do not proceed to Step 3
until the user has reviewed the analysis or explicitly waived review.

#### Step 3 — Write the paper page

Create `wiki/papers/<key>.md` using the paper template from §2. In the
frontmatter, populate the resource and license fields found in Step 0.6–0.7:

```yaml
local_paper: papers/radiance-fields/kerbl_2023_3d-gaussian-splatting.pdf
url: https://arxiv.org/abs/2308.14737
code: https://github.com/graphdeco-inria/gaussian-splatting
license_paper: arxiv-nonexclusive
license_code: non-commercial
```

In the body, include the resource + license strip at the top:
```
📄 [Full paper](../../papers/<subfolder>/<filename>) · [arXiv](https://arxiv.org/abs/XXXX.XXXXX) · [code](https://github.com/<org>/<repo>)

_Paper license: `arxiv-nonexclusive` · Code license: `non-commercial`_
```

If no code was found, replace the `[code]` link with `no code found (<date>)` so `lint find-code` has a timestamp to compare against. If a license is restrictive (non-commercial, research-only, unknown), add a `## Code & license` short section to the body calling out what downstream use it blocks — do not bury this information in frontmatter alone.

**Finding the URL**: if the user provides a URL, use it. If the paper was
dropped as a local file, search the web for the paper title + "arXiv" to find
the canonical URL. If no URL can be found, omit the `url:` field and the
arXiv link — don't guess.

#### Step 4 — Cascade updates

For each concept/method/person mentioned:
- If a page exists: update it (add a bullet, refine a claim, extend the
  lineage, flag a contradiction if the new source disagrees).
- If it doesn't exist but it's load-bearing (appears in method / results /
  related-work): create a stub with at minimum a TL;DR and the `sources:`
  backlink. Mark `status: stub`.
- Do not create a page for every name dropped — only things material to the
  paper's contribution or to an existing thread.

When referencing the paper from any wiki page, link to both the wiki
summary and the local source:
`[Kerbl et al. 2023](papers/kerbl2023_3dgs.md) · [pdf](../../papers/radiance-fields/kerbl_2023_3d-gaussian-splatting.pdf)`

#### Step 5 — Evolve affected threads (two passes)

Threads are living SOTA pipelines, not reading lists. Every ingest runs two
passes on every thread flagged by Step 2's "Pipeline contribution candidates"
(and any other thread where the new paper is clearly relevant).

**Pass A — Local, per-stage evaluation.** For each pipeline-contribution
candidate from Step 2:

1. Open the target thread. Re-read its "Goal & success criteria", "Current
   SOTA pipeline", and "Candidate components".
2. Does the new component **beat** the current SOTA pipeline's corresponding
   stage on the thread's stated success criteria (accuracy, speed, robustness,
   data requirements, assumptions)?
3. If yes: update "Current SOTA pipeline" to use the new component. Supersede
   the prior entry with strikethrough + a "superseded by" note per Hard Rule
   §6.3. Record the swap in "Pipeline lineage" with driver paper + rationale.
4. If no but the contribution is interesting: add it to "Candidate components
   / not yet integrated" with a note on *why* it lost (and under what
   conditions it might win later).
5. If the paper *contradicts* a current pipeline choice (claims it's wrong,
   not just worse): flag in "Contradictions & tensions" and require explicit
   user review before changing the pipeline.
6. If the paper opens a stage or problem no thread currently covers: propose
   (don't silently create) a new thread.

**Pass B — Holistic synthesis (mandatory, even when Pass A changed nothing).**
Step back from stage-by-stage substitution and reason about the pipeline as a
whole system. The core ambition of this project — finding *novel* combinations
of ideas that beat any single paper's pipeline — lives entirely in this pass.
For each affected thread, write answers to:

1. **Do any ideas compose?** Does this paper's contribution unlock, strengthen,
   or get unlocked by an *existing* candidate in this thread's backlog (or in
   another thread)? Example: a new uncertainty estimator may make a
   previously-shelved refinement method viable. Say so explicitly, or say
   "nothing composes" explicitly.
2. **Cross-stage interactions.** Does the new component change *assumptions*
   of upstream or downstream stages in a way that invites redesigning those
   stages too? Locally neutral swaps can be globally beneficial when they
   remove a constraint elsewhere. Locally beneficial swaps can be globally
   harmful if they break a downstream assumption.
3. **New combinations not proposed in any single paper.** Propose at least one
   *novel* pipeline variant that mixes ideas across ≥2 papers (including this
   one) in a configuration none of them proposed. State hypothesis, ideas
   combined, expected gain, risk, and what experiment would validate it. Add
   to "Open questions & synthesis bets".
4. **Cross-thread transfer.** Could an idea from this paper, or any candidate
   in this thread's backlog, improve a *different* thread's pipeline? File a
   cross-thread note in both threads if so.
5. **Verdict on the SOTA pipeline as a whole.** After the above, decide
   whether the pipeline should be revised as a *coherent whole* — not just
   one stage swapped, but possibly several stages redesigned together.
   Propose the revised pipeline, preserve the prior one via "Pipeline
   lineage", and record the cross-stage rationale.

If Pass B produces nothing — no compositions, no novel combinations, no
cross-thread transfer — state this explicitly in the ingest report. A silent
Pass B almost always means it was skipped.

Preserve prior hypotheses with strikethrough or "superseded by" — don't
silently rewrite history (Hard Rule §6.3).

#### Step 6 — Update `index.md`

Add new pages; bump `updated:` dates on touched ones.

#### Step 7 — Append to `log.md`

Use the format in §5. Include the `local_paper:` path in the log entry.

#### Step 8 — Report back

List of files created/modified (with link syntax), the local paper path,
and any contradictions or open questions the ingest surfaced.

**Budget**: a single ingest typically touches 5–15 wiki pages. More than 20 is
a smell — you're probably creating pages for trivia. Fewer than 3 is a smell
too — you're probably missing cascade updates.

A good ingest also produces:
- at least one concrete pipeline-contribution evaluation (Pass A), even if
  the verdict is "does not improve any current SOTA pipeline";
- an explicit Pass B holistic-synthesis note on every affected thread, even
  if that note concludes "no new combinations surfaced this round".

An ingest that adds a paper page without writing anything in any thread's
"Current SOTA pipeline", "Candidate components", or "Open questions &
synthesis bets" section is a smell: deep analysis (Step 2) or holistic
synthesis (Step 5 Pass B) was skipped.

### 3.2 Query

When the user asks a question:

1. **Read `index.md` first** to locate relevant pages. Then read those pages.
   Don't grep the raw sources unless the wiki clearly doesn't cover it.
   If you need to read a source paper and the local file is missing, use
   the cache-check from Step 1 of §3.1 to download it first.
2. **Answer with citations** using relative links to paper/method pages.
3. **If the wiki is insufficient**, say so explicitly: "the wiki doesn't
   cover X — want me to ingest a source on it?"
4. **Offer to file the answer back.** If the answer is substantive (a
   comparison, a synthesis, a new connection), ask: "should I save this as
   `wiki/threads/<slug>.md`?" Good answers are wiki material, not chat fluff.

### 3.3 Lint

`lint` is the single entry point for wiki health. Bare `lint` diagnoses;
`lint <action>` fixes a specific category of issues.

#### Bare `lint` — diagnose (read-only)

When the user says "lint" or "health check":

Scan the wiki and report (do not auto-fix — present a list for human approval):
- **Contradictions**: pages making incompatible claims.
- **Stale claims**: older pages whose claims a newer source has superseded.
- **Orphans**: pages with zero inbound references. A reference is **either** an Obsidian wikilink (`[[slug]]` or `[[slug|alias]]`) **or** a relative markdown link to a wiki page (`[text](../papers/<slug>.md)`, `(wiki/methods/<slug>.md)`, etc.). Per §2, concept/method/thread pages are linked as wikilinks while paper citations use relative markdown links — the detector must count both, or every paper cited only via markdown links will be falsely flagged as an orphan.
- **Missing pages**: concepts/methods referenced ≥3 times with no own page.
- **Broken links**: wikilinks pointing to non-existent pages.
- **Frontmatter drift**: missing `updated:` dates, empty `sources:`, etc.
- **Stale threads**: threads whose `updated:` predates the `updated:` date of at least one paper page in their own `sources:` list — the narrative hasn't absorbed what its own citations now say. See `lint stale-threads` for the remediation action.
- **Missing papers**: wiki pages with a `url:` field but no local file at `local_paper:`.
- **Missing code**: paper / method / dataset pages with no `code:` field *and* no "no code found (<date>)" marker in the body — these have never been searched. Also flag pages whose "no code found" marker is older than 6 months — time to re-check. Remediation: `lint find-code`.
- **Missing license**: paper pages without `license_paper:`, or with `code:` set but `license_code:` missing, or dataset pages without `license_dataset:`. Remediation: `lint licenses`.
- **Restrictive licenses**: paper / method / dataset pages whose `license_code:` is `non-commercial` / `research-only` / `unknown`, or `license_dataset:` is `non-commercial` / `custom-research` — report these as a *usability map* (which pipeline components are actually redistributable), not as a fix target. No `lint` action; this is informational.
- **Investigation suggestions**: 3–5 follow-up questions or source hunts.

#### `lint <action>` — targeted fixes

After reviewing a lint report, the user can run `lint <action>` to fix a
specific category. Each action targets exactly one diagnostic and requires
user approval before making changes.

| Command | What it fixes | Writes to |
|---------|---------------|-----------|
| `lint fetch` | Downloads all missing papers (wiki pages with `url:` but no local file). | `papers/` cache only (wiki-read-only) |
| `lint frontmatter` | Fills in missing `updated:` dates, empty `sources:` arrays, and other frontmatter drift. | `wiki/` pages |
| `lint orphans` | Proposes inbound wikilinks for orphan pages. | `wiki/` pages |
| `lint stale-threads` | Identifies threads whose narrative has fallen behind their own cited sources; proposes a targeted re-cascade. | `wiki/threads/*.md` (after per-thread approval) |
| `lint find-code` | Re-searches for code implementations for paper / method / dataset pages that currently have no `code:` field. Populates `code:` + `license_code:` on hits. | `wiki/papers/*.md`, `wiki/methods/*.md`, `wiki/datasets/*.md` (after per-page approval) |
| `lint licenses` | Fills in missing `license_paper:`, `license_code:`, `license_dataset:` on pages that have the underlying resource (`url:`, `code:`, or dataset reference) but no license recorded yet. | `wiki/papers/*.md`, `wiki/methods/*.md`, `wiki/datasets/*.md` (after per-page approval) |

New actions can be added as patterns emerge. The contract: **bare `lint` is
always safe and read-only; `lint <action>` may write but always asks first.**

#### `lint fetch` procedure

1. **Scan** all wiki paper pages (`wiki/papers/**/*.md`) and extract
   `local_paper:` and `url:` from frontmatter.
2. **Filter** to papers where `local_paper:` is set, `url:` is set, and
   the local file does **not** exist on disk.
3. **Present** a numbered list to the user:
   _"Found N papers with missing local files. Will download:
   1. `papers/radiance-fields/kerbl_2023_3d-gaussian-splatting.pdf` ← arXiv 2308.14737
   2. `papers/sfm-slam/schonberger_2016_sfm.pdf` ← arXiv 1604.03637
   …"_
4. On approval, **download** each paper:
   - Create any missing subdirectories under `papers/`.
   - arXiv URLs: `curl -fL -o <local_paper_path> https://arxiv.org/pdf/<id>`
   - Other URLs: `curl -fL -o <local_paper_path> <url>`
   - Use `-f` so HTTP errors fail the command instead of writing an HTML error page to disk.
   - If a download fails, delete any partial file at `<local_paper_path>` before moving on.
5. **Report** results: how many succeeded / failed, and list any failures
   with their URLs so the user can investigate manually (paywalled, dead link,
   moved, etc.).
6. **Append to `log.md`**.

**Notes on `lint fetch`**:
- Wiki-read-only — never creates, modifies, or deletes wiki pages.
- Papers without a `url:` field are silently skipped (listed separately in the report so the user can add URLs later).
- If all local files already present: "All N papers present — nothing to download."
- **Primary use case**: after a fresh `git clone`, run `lint fetch` to repopulate the `papers/` cache (which is git-ignored) from each wiki page's `url:` field.

#### `lint stale-threads` procedure

A thread is **stale** when its narrative hasn't kept up with its own cited sources. The check is deterministic: compare a thread's `updated:` date against the most recent `updated:` date among the paper pages it lists in `sources:`. If a thread predates any of its sources, the thread body has not absorbed whatever those papers contributed.

1. **Scan** all `wiki/threads/*.md`. For each thread, parse the `updated:` date and the `sources:` list from frontmatter.
2. **Compute staleness** for each thread:
   - `thread_updated` = thread's `updated:` date.
   - `source_newest` = `max(source.updated:)` across every paper page listed in the thread's `sources:` (resolve relative paths; skip sources that don't exist on disk and flag them separately).
   - Thread is **stale** if `thread_updated < source_newest`, with lag = `source_newest − thread_updated` days.
3. **Present** a table per stale thread:
   _"Thread `[[foundation-features-for-geometry]]` last updated 2026-04-15, lag 4 days.
   Newer sources not yet reflected:
   - `papers/chen2026_ttt3r.md` (updated 2026-04-19, added [TL;DR in 1 line])
   - `papers/zhang2025_loger.md` (updated 2026-04-18, added [TL;DR in 1 line])
   Suggest: reread these two papers and update the `### Feed-forward 3D reconstruction` and `## Outstanding hypotheses` sections."_
4. On per-thread approval, **do a targeted re-cascade** (mirroring Step 5 of §3.1 ingest): reread each newer source, update the thread body where those papers advance, contradict, or extend the existing narrative, bump `updated:`. Do **not** auto-rewrite without approval — the whole point of the stale signal is that only a human-in-the-loop synthesis is trustworthy.
5. **Report** per-thread: which sections changed, and any contradictions surfaced that warrant updates to other threads.
6. **Append to `log.md`**.

**Notes on `lint stale-threads`**:
- **Only flags real staleness, not bureaucratic staleness**: a thread whose last `updated:` is old but whose sources are also old is not stale — its narrative is caught up with what's in the wiki.
- **False negatives possible**: if a paper was ingested but never added to any thread's `sources:`, stale-threads won't catch it. That's what `lint orphans` is for.
- **Ingest hygiene complement**: after a batch ingest that touches many threads, run `lint stale-threads` to find the cascade updates you missed. Step 5 of §3.1 says to update threads during ingest — this is the safety net for when that step gets rushed.
- **Lag threshold tuning**: by default every non-zero lag counts. If that's too noisy on a large wiki, filter to threads with lag ≥ N days (user-specified) — but err on reporting more.

#### `lint find-code` procedure

Code releases lag papers by months or years — a paper ingested in 2024 with
no official code may have gained an official or canonical community
implementation by 2026. `lint find-code` re-searches for those.

1. **Scan** all `wiki/papers/**/*.md`, `wiki/methods/**/*.md`, and
   `wiki/datasets/**/*.md`. Select pages where **either**:
   - the `code:` field is absent/empty, **or**
   - the body contains `no code found (<date>)` and that date is older than
     6 months from today.
2. **For each selected page, search** in the same order as ingest Step 0.6:
   paper body mentions → project page → Papers-with-Code → GitHub search for
   `<first-author> <short-title>`. Accept only official or canonical
   community implementations — same rule as ingest.
3. **Present** a table to the user per found candidate:
   _"Page `wiki/papers/kerbl2023_3dgs.md` — candidate: `github.com/graphdeco-inria/gaussian-splatting`
   (first-author repo, 18.2k stars, last commit 2026-02-14, LICENSE: Gaussian-Splatting-License (non-commercial research only))._"
   For each, state: **repo URL · why it's canonical · LICENSE file contents (SPDX if possible) · last-commit date · star count**. The LICENSE readout is critical — an unreadable license kills the candidate.
4. **On per-page approval**, update frontmatter: set `code:`, set
   `license_code:`, update the resource strip in the body. If the repo has
   a restrictive license (non-commercial / research-only / unknown), also
   add or update the `## Code & license` body section flagging downstream
   implications.
5. **For pages with no candidate found**, refresh the `no code found
   (<date>)` marker in the body with today's date so the next
   `lint find-code` run 6 months from now has a correct timestamp.
6. **Append to `log.md`** with counts: pages searched, pages newly linked,
   pages refreshed with no-code marker, restrictive-license flags raised.

**Notes on `lint find-code`**:
- Never overwrites an existing `code:` entry — if you think the current code
  link is wrong or stale (e.g. repo deleted), that's a separate manual fix,
  not automation.
- Non-commercial licenses are valid results (3DGS, several large-model
  papers) — do not reject a canonical repo because its license is
  restrictive. Record faithfully and flag downstream.
- Respect rate limits on GitHub / Papers-with-Code; batch queries and back
  off on 429s rather than hammering.

#### `lint licenses` procedure

Separate from `lint find-code` because some papers have no code but still
need a paper license recorded, and some have code but no code-license yet.

1. **Scan** all `wiki/papers/**/*.md`, `wiki/methods/**/*.md`, and
   `wiki/datasets/**/*.md`. Select pages where:
   - `url:` is set but `license_paper:` is missing, **or**
   - `code:` is set but `license_code:` is missing, **or**
   - page type is `dataset` and `license_dataset:` is missing.
2. **Resolve each**:
   - `license_paper:` via arXiv abstract page scrape (submission license
     footer) or publisher page. If ambiguous, record `unknown` rather than
     guessing.
   - `license_code:` via the repo's `LICENSE` file (raw.githubusercontent.com
     fetch) or GitHub's `license` API endpoint. Map to SPDX when possible.
   - `license_dataset:` via the dataset's release page / README / LICENSE
     file.
3. **Present** a table of proposed fills per page, with the evidence source
   (URL, API response, file path). User can approve-all or per-page.
4. **On approval**, update frontmatter and refresh the resource strip in
   the body. Non-SPDX licenses (`arxiv-nonexclusive`, `Gaussian-Splatting-License`,
   `custom-research`) should be recorded verbatim with a short gloss in the
   body `## Code & license` section.
5. **Append to `log.md`**.

**Notes on `lint licenses`**:
- Wiki-write-only; never fetches / modifies / deletes files in `papers/`
  or `articles/` caches.
- `unknown` is a legitimate value and must remain visible — downstream
  readers need to know when a license is ambiguous rather than missing.

## 4. `index.md` format

Grouped by category. Each line:
`- [<title>](<relative-path>) — <one-line hook> · _updated YYYY-MM-DD_`

Keep it navigable by a human. If the index grows past ~300 lines, split into
sub-indexes (`index-papers.md`, `index-methods.md`) and have `index.md` link
to them.

## 5. `log.md` format

Append-only, chronological, newest at bottom. Every entry starts with a
consistent prefix so `grep "^## \[" log.md | tail -20` works:

```
## [YYYY-MM-DD] ingest | <paper short title>
- Created: wiki/papers/<key>.md, wiki/methods/<new>.md
- Updated: wiki/threads/<thread>.md (added lineage entry), wiki/methods/<x>.md (contradiction flagged)
- Pipeline impact: <thread>: <stage> updated (<old> → <new>) · <thread>: N candidates queued · <thread>: Pass B — no change warranted
- Synthesis bet: <one-line novel combination proposed, if any, and where filed>
- Notable: <one-line interesting finding or open question>

## [YYYY-MM-DD] query | <short question>
- Touched: <pages read>
- Filed as: wiki/threads/<slug>.md (if saved)

## [YYYY-MM-DD] lint
- Contradictions: 2 · Orphans: 1 · Missing pages: 3 · Stale: 0
- Followups suggested: see log entry body

## [YYYY-MM-DD] lint fetch
- Downloaded: N papers (M already present, K failed)
- Failures: <list of failed URLs if any>

## [YYYY-MM-DD] lint find-code
- Pages searched: N (P papers + M methods + D datasets)
- Newly linked: K pages (list top 3 or so: `wiki/papers/<key>.md` → `<repo>` [license])
- No-code-marker refreshed: R pages
- Restrictive licenses flagged: <count> (list categories: non-commercial / research-only / unknown)

## [YYYY-MM-DD] lint licenses
- Filled: N paper-licenses, M code-licenses, D dataset-licenses
- Unknowns retained: K (list brief reasons if interesting)
```

## 6. Hard rules

1. **Never edit source papers.** Files in `papers/` and `raw/` are read-only
   after placement (renaming/moving during ingest is allowed, content edits are not).
2. **Never invent citations.** If you don't have the source, mark `[!needs-source]`.
3. **Never silently overwrite a contradicted claim.** Preserve the prior
   position with strikethrough or a "superseded by" note and link to the new source.
4. **Stay in scope.** Photogrammetry + ML research. If the user drops something
   off-topic, ask before filing.
5. **Don't over-create pages.** A name-drop is not a page. A page must have a
   defensible reason to exist (referenced by ≥2 other pages, or materially
   advances a thread).
6. **Cite down to the page.** When making a factual claim in a wiki page,
   the `sources:` frontmatter must include the paper(s) that back it.
7. **Obsidian is the read-side UI.** Wikilinks, frontmatter, and relative paths
   should all "just work" in Obsidian without plugins beyond Dataview.
8. **No emojis in wiki content** unless the user asks.

## 7. Co-evolution

This file is not frozen. When the user develops a workflow that works (or
catches one that doesn't), update this schema. Log schema changes under a
`## [YYYY-MM-DD] schema-change` entry in `log.md`.

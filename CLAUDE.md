# Vision Wiki — Schema & Operating Manual

This is a **personal second brain** for photogrammetry and machine-learning research.
You (the LLM) are the sole author and maintainer of the `wiki/` layer. The human
curates sources and asks questions. Never invent citations.

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
├── papers/                # permanent store: research papers, PDFs (see §1.2)
│   ├── radiance-fields/   # example subfolder — see §1.2 for taxonomy
│   ├── feature-matching/
│   └── ...
├── articles/              # permanent store: blog posts, web articles, notes (see §1.3)
├── assets/                # permanent store: images, figures, diagrams (see §1.4)
└── wiki/                  # LLM-authored, constantly-maintained knowledge base
    ├── papers/            # one page per ingested paper (the "source summary")
    ├── methods/           # algorithms, architectures, techniques (e.g. `3d-gaussian-splatting.md`)
    ├── concepts/          # general ideas and primitives (e.g. `bundle-adjustment.md`)
    ├── datasets/          # benchmark + training datasets
    ├── people/            # prolific authors / research groups
    └── threads/           # evolving syntheses, open questions, comparisons
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

The top-level `papers/` directory holds all collected source papers (PDFs,
markdown clippings, arXiv HTML exports). The LLM manages this folder — it
downloads, renames, and organizes papers here.

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
type: paper | method | concept | dataset | person | thread
tags: [nerf, differentiable-rendering, ...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [papers/kerbl2023_3dgs.md, papers/mildenhall2020_nerf.md]
status: stub | draft | stable | contested
---
```

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

**Thread page** (`wiki/threads/<name>.md`): an *evolving synthesis* — e.g.
`radiance-field-evolution.md`, `feature-matching-learned-vs-classical.md`.
Hypothesis at top, supporting evidence below, contradictions flagged, open
questions at bottom. Reread and revise on every relevant ingest.

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
6. Report the final path to the user (e.g. "Saved to
   `papers/radiance-fields/kerbl_2023_3d-gaussian-splatting.pdf`").

#### Step 1 — Read the source

Read the source fully. For PDFs, read all pages. For arXiv markdown,
read the whole file. Don't skim.

#### Step 2 — Discuss before writing

Report TL;DR (2–3 sentences), the 3 most important takeaways, and what
existing wiki pages this will touch. Ask the user if they want to
emphasize anything before you commit edits.

#### Step 3 — Write the paper page

Create `wiki/papers/<key>.md` using the paper template from §2. In the
frontmatter, add a `local_paper:` field pointing to the local file:

```yaml
local_paper: papers/radiance-fields/kerbl_2023_3d-gaussian-splatting.pdf
```

In the body, include a link to the local paper at the top:
`📄 [Full paper](../../papers/<subfolder>/<filename>)`

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

#### Step 5 — Update affected threads

If the paper advances or contradicts a thread, revise the thread page.
Preserve prior hypotheses with strikethrough or a "superseded by" note —
don't silently rewrite history.

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

### 3.2 Query

When the user asks a question:

1. **Read `index.md` first** to locate relevant pages. Then read those pages.
   Don't grep the raw sources unless the wiki clearly doesn't cover it.
2. **Answer with citations** using relative links to paper/method pages.
3. **If the wiki is insufficient**, say so explicitly: "the wiki doesn't
   cover X — want me to ingest a source on it?"
4. **Offer to file the answer back.** If the answer is substantive (a
   comparison, a synthesis, a new connection), ask: "should I save this as
   `wiki/threads/<slug>.md`?" Good answers are wiki material, not chat fluff.

### 3.3 Lint

When the user says "lint" or "health check":

Scan the wiki and report (do not auto-fix — present a list for human approval):
- **Contradictions**: pages making incompatible claims.
- **Stale claims**: older pages whose claims a newer source has superseded.
- **Orphans**: pages with zero inbound wikilinks.
- **Missing pages**: concepts/methods referenced ≥3 times with no own page.
- **Broken links**: wikilinks pointing to non-existent pages.
- **Frontmatter drift**: missing `updated:` dates, empty `sources:`, etc.
- **Thread debt**: threads not updated in the last N ingests that touched them.
- **Investigation suggestions**: 3–5 follow-up questions or source hunts.

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
- Notable: <one-line interesting finding or open question>

## [YYYY-MM-DD] query | <short question>
- Touched: <pages read>
- Filed as: wiki/threads/<slug>.md (if saved)

## [YYYY-MM-DD] lint
- Contradictions: 2 · Orphans: 1 · Missing pages: 3 · Stale: 0
- Followups suggested: see log entry body
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

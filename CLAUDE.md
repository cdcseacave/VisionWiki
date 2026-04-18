# Vision Wiki — Schema & Operating Manual

This is a **personal research-synthesis engine** for photogrammetry and
machine-learning. It is not a reading list and not a summarization tool. Its
end-product is **novel pipelines**: each thread maintains the current
best-known end-to-end pipeline(s) for a problem — one per *operating point*
— and every ingested paper is mined for ideas that could improve those
pipelines, either by replacing a stage or by unlocking a *new combination*
of ideas that no single paper has tried. Deep mechanism-level understanding
of each paper, and holistic reasoning about how ideas compose across
papers, are the two activities the wiki exists to support.

You (the LLM) are the sole author and maintainer of the `wiki/` layer. The
human curates sources and asks questions. Never invent citations. A shallow
summary is worse than useless — if you can't explain the mechanism of a new
idea, you don't yet understand the paper.

Domain focus: photogrammetry, SfM, MVS, SLAM, NeRF / 3D Gaussian Splatting,
neural implicit surfaces, feature matching, bundle adjustment, camera
calibration, point-cloud / mesh processing, depth estimation, pose
estimation, and the ML methods that power them (transformers, diffusion,
self-supervised learning, etc.).

---

## 1. Directory layout

```
VisionWiki/
├── CLAUDE.md              # this file — the schema
├── index.md               # content-oriented catalog of every wiki page
├── log.md                 # chronological append-only activity log
├── raw/                   # INBOX — drop anything here, emptied on ingest (§1.1)
├── papers/                # LOCAL CACHE (git-ignored): PDFs, re-downloadable via url: (§1.2)
│   ├── radiance-fields/
│   ├── feature-matching/
│   └── ...
├── articles/              # permanent store: blog posts, web articles, notes (§1.3)
├── assets/                # LOCAL CACHE (git-ignored): images, figures, diagrams (§1.4)
└── wiki/                  # LLM-authored, constantly-maintained knowledge base
    ├── papers/            # one page per ingested paper (the "source summary")
    ├── ideas/             # atomic novel contributions extracted from papers (§1.6)
    ├── stages/            # typed pipeline stages — input/output/invariant schemas (§1.7)
    ├── methods/           # algorithms, architectures, techniques
    ├── concepts/          # general ideas and primitives
    ├── datasets/          # benchmark + training datasets
    ├── people/            # prolific authors / research groups
    ├── threads/           # evolving per-goal pipelines
    └── designs/           # concrete "how to build it" plans
```

### 1.1 Inbox (`raw/`)

`raw/` is a **drop zone / inbox**. The user dumps any files here — PDFs,
markdown clippings, images, screenshots — with whatever messy names they
have. The LLM never reads directly from `raw/` during queries.

On ingest (bare `ingest` with no arguments), the LLM:
1. Scans `raw/` recursively for all files.
2. Classifies each file by type (paper, article, or asset) — §1.5.
3. Renames and moves it to the appropriate permanent store.
4. Deletes the original from `raw/`.

**After a complete ingest, `raw/` should be empty.** A non-empty `raw/`
is the primary signal that work remains.

### 1.2 Paper storage (`papers/`)

`papers/` is a **local cache** of source PDFs / markdown / arXiv HTML.
Git-ignored (PDFs are large + binary). Wiki pages in `wiki/papers/` are
committed; their `url:` field lets any clone re-download on demand.

**Cache behavior**: if the local file at `local_paper:` is missing but
`url:` exists, download it first:
- arXiv: `curl -L -o <local_paper_path> https://arxiv.org/pdf/<id>`
- Other: `curl -L -o <local_paper_path> <url>`

**Naming**: `<FirstAuthor>_<Year>_<Short-Title>.<ext>` (lowercase
kebab-case short title). E.g. `kerbl_2023_3d-gaussian-splatting.pdf`.

**Subfolder taxonomy** — create / merge / restructure as volume demands:

| Subfolder | Scope |
|-----------|-------|
| `radiance-fields/` | NeRF, 3DGS, neural implicit surfaces, novel-view synthesis |
| `feature-matching/` | keypoint detection, descriptor learning, matching |
| `sfm-slam/` | structure from motion, visual SLAM, visual odometry |
| `mvs-depth/` | multi-view stereo, mono/stereo depth |
| `pose-estimation/` | camera / object pose, PnP |
| `mesh-reconstruction/` | surface reconstruction, meshing, point-cloud |
| `fundamentals/` | general ML methods, transformers, diffusion, optimization |
| `datasets-benchmarks/` | dataset papers, benchmark comparisons |

If a paper spans categories, file under its **primary contribution** and
note cross-topic relevance in the wiki page.

### 1.3 Article storage (`articles/`)

Blog posts, tutorials, non-paper text sources. Renamed on ingest to
`<source>_<year>_<short-title>.md`. Same subfolder taxonomy as `papers/`
where applicable.

### 1.4 Asset storage (`assets/`)

Images, figures, diagrams. Renamed to
`<paper-or-article-key>_<descriptor>.<ext>`. Referenced as
`![caption](../assets/<file>)`. Never hot-link external URLs.

### 1.5 Classification rules for ingest

| File type | Destination |
|-----------|-------------|
| PDF (research paper) | `papers/<subfolder>/` |
| Markdown / HTML (blog / tutorial) | `articles/<subfolder>/` or `articles/` |
| Image (PNG, JPG, SVG) | `assets/` |
| PDF (slide deck, report) | `articles/` |
| Unknown / ambiguous | Ask the user before filing |

### 1.6 Ideas (`wiki/ideas/`)

Ideas are the atomic unit of synthesis. Every distinct novel contribution
identified in ingest Step 2 becomes a first-class page at
`wiki/ideas/<slug>.md`. Threads, designs, and synthesis bets reference
ideas by wikilink — composition questions ("does X compose with Y?",
"what unlocks stage Z?") become graph queries, not re-derivation from
prose.

Each idea has structured frontmatter declaring its mechanism, target
stage(s), I/O types, assumptions, and composition edges (`requires:`,
`unlocks:`, `co_requires:`, `refines:`, `equivalent_to:`, `contradicts:`).
See §2.1 Idea page for the template and scope enum.

**Ideas are not always 1:1 stage fillers.** The `scope:` field declares
whether the idea is drop-in, collapse of several stages, split, topology
rewrite, new paradigm, or bridge adapter. Ideas that only work jointly
declare this via `co_requires:` — bundle truth is a property of the idea,
not of any specific bet.

Naming: `<short-descriptor>_<firstauthor><year>.md`, e.g.
`anisotropic-gaussians_kerbl2023.md`. Mechanism-focused, not
marketing-focused.

### 1.7 Stages (`wiki/stages/`)

A stage is a typed slot in a pipeline — e.g. `radiance-fields.rendering`,
`sfm.initial-pair-selection`. Stage pages declare input/output types,
invariants, and data regime. Thread SOTA pipelines are DAGs of typed
stages, filled by ideas whose `stages:` field matches the slot. This
makes composition type-checkable: an idea producing
`unposed-images → posed-images` cannot fill a slot requiring
`posed-images → radiance-field`.

Stage IDs are dotted: `<domain>.<slot>`. Create stage pages
opportunistically — when a thread references an unbound slot, or when an
idea's `stages:` introduces a new one. Stage pages stay small
(≤ half a page); they exist to make the type system legible.

## 2. Page conventions

Every wiki page has YAML frontmatter:

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
license_paper: CC-BY-4.0                         # paper pages only; "unknown" if undeterminable
license_code: MIT                                # paper / method / dataset; "unknown" if repo has no LICENSE
license_dataset: CC-BY-NC-4.0                    # dataset pages only
status: stub | draft | stable | contested
---
```

Type-specific extensions (`operating_points:` on threads, `scope:` /
`stages:` / composition edges on ideas, `realizes_bet:` on designs) are
defined in §2.1.

**Code & license**:
- `code:` must be **official** (authors' repo) or **canonical community**
  (the most-cited community port when authors released none). Do not
  link random forks. If search finds nothing, omit and write
  `no code found (<date>)` in the body — `lint find-code` re-checks.
- `license_paper:` / `license_code:` / `license_dataset:` — SPDX when
  possible (`CC-BY-4.0`, `MIT`, `Apache-2.0`), otherwise verbatim with a
  gloss (`arxiv-nonexclusive`, `Gaussian-Splatting-License`,
  `custom-research`). **Flag non-commercial / research-only / unknown
  explicitly in the page body** under `## Code & license` — these
  materially affect downstream usability.

**`sources:` semantics**:
- **method / concept / thread / dataset / person**: paper pages that
  back claims. Required, non-empty.
- **paper**: the page *is* the source. Omit, or list prior work the
  paper builds on. Empty `sources: []` is not a lint violation.
- **design**: wiki pages (any type) the design rests on.

**Filenames, cross-links, math, figures, citations**:
- Filenames lowercase-kebab-case. Papers: `<firstauthor><year>_<slug>.md`
  (e.g. `kerbl2023_3dgs.md`). Methods/concepts: canonical name.
- Cross-links: Obsidian wikilinks `[[slug]]` for
  concept/method/person/thread. Relative markdown links for paper
  citations: `[Kerbl et al. 2023](papers/kerbl2023_3dgs.md)`.
- Math: inline `$...$`, display `$$...$$` (MathJax).
- Figures: `![caption](../assets/<file>)`. No external hot-linking.
- Every factual claim must trace to `sources:`. Unsourceable → mark
  `> [!needs-source]`.

### 2.1 Page templates

#### Paper page (`wiki/papers/<key>.md`)

```markdown
📄 [Full paper](../../papers/<subfolder>/<filename>.pdf) · [arXiv](https://arxiv.org/abs/XXXX.XXXXX) · [code](https://github.com/<org>/<repo>)

_Paper license: `<spdx>` · Code license: `<spdx>`_   <!-- omit code half if no code; "no code found (<date>)" if searched -->

## TL;DR
One paragraph. What is the contribution in one breath?

## Problem
What limitation of prior work does this address?

## Method
Core technical idea. Math where it clarifies. Pseudocode if useful.

## Results
Headline numbers, datasets, comparisons. Only numbers that appear in the paper.

## Why it matters
How this fits into the broader [[thread-name]] and what it enables downstream.

## Pipeline contribution
Wikilinks to the idea pages produced in ingest Step 3 — one bullet per idea:

- [[idea-slug]] · candidate thread: [[thread-slug]] · op_target: `op:<name>` (if thread has >1) · one-line mechanism recap · expected gain.

Mechanism lives on the idea page; this section is cross-reference only.

## Relation to prior work
- Builds on [[method]] / [Author Year](papers/...)
- Contrasts with [[method]]

## Open questions / limitations
Both the paper's stated limitations and your own skepticism.

## Code & license
Only when non-commercial / research-only / unknown. State what downstream use it blocks.

## References added to the wiki
Pages created or meaningfully updated by this ingest.
```

#### Method page (`wiki/methods/<name>.md`)

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

#### Thread page (`wiki/threads/<name>.md`)

A thread is a living per-goal pipeline, not a reading list. **Every new
thread triggers the goal-proposal workflow in §3.3** — the LLM drafts
`## Goal` + optional `## Goal contract` from seed papers and presents
both to the user for edit before commit.

Frontmatter:

```yaml
---
title: ...
type: thread
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [papers/...]
operating_points: [op:default]        # 1–3 entries; cap at 3 (Hard Rule §6.9)
status: stub | draft | stable | contested
---
```

Body:

```markdown
## Goal
One-paragraph plain-language goal. Broad is fine — e.g. "reconstruct the
best possible mesh from a 3DGS scene." Required.

## Goal contract (optional, structured)
```yaml
metric: [chamfer-mm, normal-consistency, fps@inference]
target_regime: [posed | unposed, sparse-view | dense-view, bounded | unbounded]
constraints: [no-per-scene-fitting, ≤1-A100-train, commercial-license-ok]
required_capabilities: [view-synthesis, geometry-extraction, relighting]
```
Filled when known. Lints needing a contract skip threads without one.

## SOTA pipelines

One subsection per operating point. With only `op:default`, a single
flat section is fine.

### op:<name> (short qualifier — e.g. "offline, ≥1 GPU-hour")

The pipeline is a DAG of **nodes**. Each node has a thread-local stable
ID (`n1`, `n2`, ...); IDs never reused — a deleted node's ID retires.

Format:
- **n<id>** · stage(s): `[[<domain>.<slot>]]` (≥1 wikilink) · filler: `[[idea-slug]]` · source: `[Author Year](wiki/papers/<key>.md)` · gain over prior: `<specific benchmark / ablation>` · upstream: `[n<id>, ...]` · downstream: `[n<id>, ...]` · edge types: `<in: X, Y; out: Z>`.

- **Composite nodes** (`scope: multi-stage-collapse`): list every stage in `stages:`, not just one.
- **Bundle nodes** (filler has `co_requires:`): every co_required idea must appear as the filler of some node in *this* pipeline (§6.14).
- **Bridge nodes** (`scope: bridge`): marked `(bridge)` after the ID; one-line rationale for the neighbor-mismatch they resolve.

## Pipeline lineage
Chronological record of swaps and redesigns, per OP:

- **filler-swap**: `YYYY-MM-DD · op:<name> · filler-swap · n<id>: [[old-idea]] → [[new-idea]]` · driver: [paper link] · gain: `<benchmark>` · rationale: ...
- **topology-change**: `YYYY-MM-DD · op:<name> · topology-change · scope: <multi-stage-collapse | stage-split | topology-rewrite | new-paradigm> · nodes removed: [n<id>, ...] · nodes introduced: [n<id>, ...] · edges rewired: [n<id>→n<id>, ...]` · driver · rationale.

Prior pipelines preserved (strikethrough or collapsed `<details>` block)
— never silently rewritten (§6.3). When a topology-change lands, the
superseded sub-DAG is preserved as a collapsed `<details>` block below
the new pipeline.

## Candidate components / not yet integrated
Promising ideas that didn't beat SOTA (on any OP). Each: `[[idea-slug]]`
· source paper · target stage · OP considered · why it lost · what
condition could make it win.

## Open questions & synthesis bets
Structured queue, not prose. See §2.2 for the bet template. This is the
research agenda — keep it alive.

## Capability gaps
What the SOTA pipeline(s) lack. Each gap = capability + which bets it
would unlock + what to look for next ingest. The shopping list for
ingest. Refreshed every Pass B (§3.1 Step 5).

- **<capability>** — would unlock Bets #NNN, #MMM. Search target: <what kind of paper supplies this>.

## Contradictions & tensions
Where ingested papers disagree. Current resolution and confidence.

## Shelved bets / known non-compositions
Bets refuted / superseded / infeasible. Each: `Bet #NNN` · final status ·
short reason · `triggers:` that would revive it. Pass B consults this
before emitting new bets.

## Sources
(populated from frontmatter `sources:`)
```

Reread on every relevant ingest. If an ingest touches a thread without
updating at least one of {SOTA pipelines, Candidate components, Open
questions & synthesis bets, Capability gaps} (or explicitly saying "no
change warranted"), synthesis was skipped.

#### Idea page (`wiki/ideas/<slug>.md`)

Frontmatter (structured and lint-parsed):

```yaml
---
title: <short human-readable name>
type: idea
source_paper: wiki/papers/<key>.md
also_in: [wiki/papers/<key>.md, ...]

# pipeline shape
scope: drop-in | stage-swap | multi-stage-collapse | stage-split | topology-rewrite | new-paradigm | bridge
stages: [<domain>.<slot>, ...]             # ≥1 stage this idea fills or affects
collapses: [<domain>.<slot>, ...]          # stages subsumed (scope: multi-stage-collapse)
splits_into: [<new-stage-slug>, ...]       # sub-stages introduced (scope: stage-split)
rewrites: {replaces: [...], introduces: [...]}   # scope: topology-rewrite

# type contracts
inputs: [...]
outputs: [...]
assumptions: [static-scene, known-intrinsics, bounded-volume]   # see §7 conflict registry
requires_upstream_property: [...]
requires_downstream_property: [...]
learned_params: [...]
failure_modes: [...]

# composition graph
requires: [idea-slug, ...]
unlocks:  [idea-slug, ...]
co_requires: [idea-slug, ...]               # bundle truth
bridges: [idea-slug, ...]
equivalent_to: []
refines: [idea-slug]
contradicts: [idea-slug]

# housekeeping
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: unclaimed | adopted-by:<thread-slug>:<op> | shelved | refuted
validated_in: [design-slug, ...]            # populated when a design built on this idea closes out
---
```

**`scope:` enum** — pick the smallest honest scope:

| scope | meaning |
|---|---|
| `drop-in` | same stage, same types, strictly better |
| `stage-swap` | same stage, compatible types, different mechanism |
| `multi-stage-collapse` | one idea subsumes ≥2 adjacent stages (list in `collapses:`) |
| `stage-split` | decomposes one stage into sub-stages (list in `splits_into:`; create stage pages) |
| `topology-rewrite` | replaces an M-node subgraph with an N-node subgraph (use `rewrites:`) |
| `new-paradigm` | changes thread goal/representation such that most prior stages no longer apply |
| `bridge` | adapter whose sole job is to convert one stage's output into another's input |

Coercing a collapse into a stage-swap is the most common schema-integrity
failure — flag rather than flatten. `co_requires:` is a property of the
idea (not of any specific bet); bets inherit it.

Body:

```markdown
## Mechanism
One paragraph. What gradient flows through what? What is frozen? What
invariance is imposed by which construction? If you can't fill this
without re-reading the paper, you haven't understood the idea yet.

## Why it wins
Causal story + specific ablation / table / number isolating this idea's
contribution. If no ablation isolates it, say so — evidence is weaker
and downstream bets must acknowledge it.

## Preconditions & compatibility
Upstream outputs consumed, downstream assumptions imposed, known
compositions (→ `unlocks:` / `requires:`), bundle constraints
(→ `co_requires:`).

## Pipeline-shape implications
Required for `scope` ∈ {multi-stage-collapse, stage-split,
topology-rewrite, new-paradigm}. State which nodes merge/split/rewire
and the resulting subgraph. Adopting threads must match this topology.

## Trade-offs vs. the decomposed pipeline
Required for `multi-stage-collapse` and `topology-rewrite`. What did the
decomposed pipeline offer that the fused one loses? (Per-stage
interpretability, ability to swap an internal stage, failure-mode
isolation, mid-stage priors, graceful degradation.) Without this
section a fused idea is systematically over-valued.

## Open questions
Things the paper did not answer that any adopting thread will have to.
```

#### Stage page (`wiki/stages/<slug>.md`)

```yaml
---
title: <human name>
type: stage
slug: <domain>.<slot>
consumes: [type, ...]                      # direct I/O input types
produces: [type, ...]                      # direct I/O output types
invariants: [scale, permutation-of-views, ...]
provides_properties: [metric-scale, per-pixel-uncertainty, ...]   # nonlocal properties the output carries
requires_upstream_properties: [...]        # nonlocal properties the stage assumes on inputs
data_regime: [bounded-scene, posed | unposed, static | dynamic, ...]
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

Body: short description, example filler ideas, notes on valid fillers.
Keep under half a page. `consumes:` / `produces:` are direct I/O types;
`provides_properties:` / `requires_upstream_properties:` are nonlocal
contracts catching the failure mode where types match but an upstream
assumption is silently broken.

#### Design page (`wiki/designs/<slug>.md`)

```yaml
---
title: ...
type: design
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [...]
realizes_bet: Bet #042                       # the bet this design implements
realizes_ideas: [[idea-a]], [[idea-b]]       # ideas this design validates
outcome: pending | succeeded | failed | partial
outcome_date: YYYY-MM-DD
status: draft | in-progress | tested | archived
---
```

Body is implementation-specific. See `wiki/designs/` for examples.
When `outcome:` flips off `pending`, also update the source bet's
`outcome:` + `status:` and each realized idea's `validated_in:`.
`lint design-closure` catches drift.

### 2.2 Synthesis bet lifecycle

Threads' "Open questions & synthesis bets" is a typed queue:

```markdown
### Bet #042 — <short hypothesis>
status: proposed | feasible | in-design | tested | validated | refuted | superseded
combines: [[[idea-a]], [[idea-b]], ...]     # ≥2 ideas, cross-paper
stage_target: <domain>.<slot>
op_target: op:<name>                        # which operating point this bet aims at
confidence: low | med | high                # likelihood it works
magnitude:  incremental | substantial | paradigm
cost:       hours | days | weeks | months
breakage_risk: low | med | high
hypothesis: ...
expected_gain: ... (tie to a benchmark)
risk: ...
validating_experiment: ...
triggers: [ingest-of-idea:<slug>, benchmark:<name>, ...]
design: wiki/designs/<slug>.md              # populated at status >= in-design
outcome: ...                                # populated at tested/validated/refuted
created: YYYY-MM-DD · updated: YYYY-MM-DD
```

When a bet reaches `feasible`, producing a design page is mandatory
before promoting to `in-design`. When an idea is ingested whose slug
appears in a shelved bet's `triggers:`, `lint unlocks-fired` surfaces
it. `lint bets` sorts the queue by
`magnitude × confidence ÷ cost ÷ breakage_risk`; bets missing rubric
fields are flagged.

## 3. Workflows

### 3.1 Ingest

The user triggers an ingest in four ways:

| Invocation | Behavior |
|------------|----------|
| **`ingest`** | **Batch mode.** Scan `raw/` for all unprocessed files; process each through Steps 0–8. |
| **`ingest <URL>`** | Download, then Steps 0–8. |
| **`ingest <local-path>`** | Process local file through Steps 0–8. |
| **`ingest papers/<subfolder>/<file>`** | Re-ingest stored paper (skip Step 0 download/rename). |

#### Step 0 — Acquire the paper(s)

**Batch mode**: list all files in `raw/`; classify per §1.5; present
numbered plan ("Found N files. 1. <name> → papers/<sub>/..."); on
approval, rename + move each per §1.2/§1.3/§1.4. Images just move.
After batch, `raw/` must be empty.

**Single-paper acquisition**:

| Input | Action |
|-------|--------|
| **arXiv URL** | `curl -L` PDF; if `/html/<id>` exists, prefer it as markdown. |
| **Direct PDF URL** | `curl -L`. |
| **Project page / blog** | WebFetch. Follow PDF links. |
| **Local file in `raw/`** | Process in place (moves out after). |
| **Local file elsewhere** | No download. |

After acquiring, do all of:

1. Determine canonical title, first author, year from content / arXiv
   metadata.
2. Classify per §1.5.
3. Rename per §1.2/§1.3/§1.4.
4. Move to the permanent store (create subfolders as needed).
5. If source came from `raw/`, the move empties it. Verify clean.
6. **Find code** (papers/datasets). Search order: paper body → project
   page → Papers-with-Code → GitHub search `<first-author> <short-title>`.
   Accept only **official** or **canonical community** repos. If
   nothing found, omit `code:` and write `no code found (<today>)` in
   the body — do not guess.
7. **Identify licenses**. `license_paper:` from arXiv footer / publisher
   page / PDF first page. `license_code:` from `LICENSE` file or GitHub
   badge. `license_dataset:` from release page. SPDX when possible,
   verbatim + gloss otherwise. Non-commercial / research-only / unknown
   → flag in body under `## Code & license`.
8. Report path + code + licenses.

#### Step 1 — Read the source

Cache check: if `local_paper:` missing but `url:` set, download first
(curl per §1.2). Read fully — for PDFs, all pages; for markdown, the
whole file. Don't skim.

#### Step 2 — Deep analysis (hard gate before writing)

Shallow summaries pollute the wiki and block synthesis. Produce this
analysis *before* writing any file and present to the user:

1. **Goal.** What problem in the paper's own framing? What does "solved"
   look like?
2. **Foundations.** Which prior methods / concepts / datasets does it
   rest on? Link every foundation to an existing wiki page; flag gaps
   where a load-bearing foundation has no page (candidate for a Step 4
   stub).
3. **Novel contributions — enumerated as idea candidates.** Each
   enumerated item is a candidate `wiki/ideas/<slug>.md` for Step 3:
   - **Proposed slug** (`<descriptor>_<firstauthor><year>`).
   - **What it is** (one line).
   - **Mechanism** — not "they use contrastive loss" but *what* vs
     *what*, under *which* invariance, driven by *which* signal. Math
     or pseudocode where it clarifies. If you can't explain the
     mechanism, you haven't understood the paper.
   - **Structured fields draft**: `scope`, `stages`, `inputs`,
     `outputs`, `assumptions`, `requires_upstream_property`,
     `requires_downstream_property`, `learned_params`, `failure_modes`,
     `co_requires`. Missing fields allowed but noted as open questions
     on the idea page.
   - **Scope classification** (load-bearing). Pick the smallest honest
     scope. Subsumes prior stages → `multi-stage-collapse` (fill
     `collapses:`). Rewires → `topology-rewrite` (fill `rewrites:`).
     Coercing a collapse into a stage-swap is the most common
     schema-integrity failure — flag rather than flatten.
   - **Bundle detection.** Relies on another idea (same paper or
     elsewhere) not yet absorbed? List under `co_requires:`.
   - **Bridge detection.** `outputs:` don't match the next stage's
     `consumes:`? Retype the idea, retype the stage, or propose a
     first-class bridge idea. Never hand-wave.
   - **Equivalence / refinement check.** Does this idea already exist
     in `wiki/ideas/`? If yes, extend `also_in:` / `refines:` rather
     than duplicate.
4. **Why it wins.** For each contribution: causal story + specific
   ablation / table / number isolating the claim. If no ablation
   isolates it, say so — evidence is weaker and worth flagging.
5. **Relation map.** Each contribution extends / replaces / contradicts
   / is orthogonal to which existing wiki pages? Cite them.
6. **Pipeline contribution candidates.** For each contribution, which
   existing thread's SOTA pipeline could absorb it, at which stage,
   at which **operating point** (if the thread has >1), and replacing
   what? If no thread owns the stage, propose whether a new thread
   should exist — this may trigger §3.3 thread creation.

Present to the user. Ask for anything to emphasize / reframe / deepen
before writing. Do not proceed to Step 3 until the user approves or
waives review.

#### Step 3 — Write paper page + idea pages + any new stage pages

First, `wiki/papers/<key>.md` using the §2.1 paper template. Then for
each enumerated idea from Step 2:

1. **No equivalent idea exists** → create `wiki/ideas/<slug>.md` per
   §2.1. Populate all structured frontmatter (leave `requires:` /
   `unlocks:` as `[]` — Pass B fills them). Set `status: unclaimed`.
2. **Equivalent exists** → add this paper to `also_in:`, extend
   `failure_modes:` / `assumptions:` if this paper surfaces new ones,
   bump `updated:`. If strictly better, create a new idea page with
   `refines: [old-slug]`.
3. **Idea references an unbound stage slug** → create a stage stub at
   minimum (consumes, produces, invariants, one-line description).
   Orphaned stage IDs break the type system.

The paper page's "Pipeline contribution" lists wikilinks to the
created/updated idea pages — mechanism lives on the idea, paper page is
source record. Populate frontmatter (`local_paper`, `url`, `code`,
`license_paper`, `license_code`) from Step 0.6–0.7. Include the resource
+ license strip at the top of the body (template §2.1).

If a license is restrictive (non-commercial / research-only / unknown),
add `## Code & license` calling out what downstream use it blocks.

**Finding the URL**: if the user provided one, use it. If the paper was
a local drop, search the web for title + "arXiv" for the canonical URL.
No guessing — omit if not found.

**Hard rule**: every novel contribution from Step 2 must produce or
update an idea page here — no back-fill escape hatch (§6.11).

#### Step 4 — Cascade updates

For each concept / method / person mentioned:
- Page exists → update (bullet, claim refinement, lineage, contradiction
  flag).
- Load-bearing and missing (appears in method / results / related-work)
  → create a stub (TL;DR + `sources:` backlink + `status: stub`).
- Not load-bearing → skip. A name-drop is not a page.

Cross-reference format:
`[Kerbl et al. 2023](papers/kerbl2023_3dgs.md) · [pdf](../../papers/radiance-fields/kerbl_2023_3d-gaussian-splatting.pdf)`.

#### Step 5 — Evolve affected threads (two passes)

For every thread flagged in Step 2 (and any other clearly relevant
thread), run both passes.

**Pass A — per-OP, per-stage evaluation.** For each idea created or
updated in Step 3:

1. Find threads whose SOTA pipeline contains `idea.stages`. Open each.
   Re-read Goal, Goal contract (if present), SOTA pipelines, Capability
   gaps, Candidate components.
2. **Type + contract check**: does `idea.inputs/outputs` match stage
   `consumes/produces`? Do upstream `provides_properties:` satisfy
   `idea.requires_upstream_property:`? Do downstream satisfy
   `requires_downstream_property:`? Mismatch → retype the idea, swap
   neighbors (defer to Pass B), or insert a bridge idea.
3. **Assumption check** (§7 conflict registry): idea's `assumptions:`
   must not conflict with the union of already-adopted assumptions on
   any OP where you're considering adoption.
4. **Scope branch**:
   - `drop-in` / `stage-swap` → single node re-filled.
   - `multi-stage-collapse` → merge nodes into a composite; record a
     topology-change lineage entry.
   - `stage-split` → decompose target into sub-stages before filling.
   - `topology-rewrite` → apply `rewrites: {replaces, introduces}`
     exactly.
   - `new-paradigm` → propose (don't silently create) a new thread or
     a parallel variant; trigger §3.3.
   - `bridge` → adopt only to resolve a specific neighbor-mismatch;
     note which neighbors.
5. **Bundle check**: `co_requires:` ideas must all be adoptable in this
   pipeline (already present or promotable together). Partial adoption
   is a type-check failure (§6.14).
6. **Per-OP evaluation**: for each operating point of the thread, does
   the new idea **beat** the SOTA pipeline's corresponding region on
   that OP's success criteria (metric from Goal contract, or Goal
   qualitative target)? An idea may win on one OP and lose on others —
   file to the winning OP only. For `multi-stage-collapse`, consult
   "Trade-offs vs. the decomposed pipeline" — a headline win that
   loses a load-bearing decomposed capability is not automatically
   adopted; flag for user review.
7. **If wins on OP X**: update that OP's SOTA pipeline to reference
   `[[idea-slug]]`. Apply the scope-appropriate DAG change. Supersede
   prior entries with strikethrough + "superseded by" (§6.3). Set
   idea's `status: adopted-by:<thread-slug>:<op>`; same on every
   `co_required` idea that travels with it. Record a lineage entry
   prefixed with `op:<name>`.
8. **If doesn't win**: add to Candidate components as `[[idea-slug]]`
   with note on *why* it lost and which OP was evaluated. If failed on
   bundle-incompleteness, list missing siblings — natural `triggers:`
   for future bets.
9. **If contradicts a current choice**: flag in Contradictions &
   tensions; require user review. Set `contradicts:` on the idea page.
10. **If opens an uncovered stage/problem**: propose a new thread
    (don't silently create — trigger §3.3). Add a stage page if
    warranted.

**Pass B — holistic synthesis (mandatory, even when Pass A changed
nothing).** The project's core ambition — novel combinations that beat
any single paper's pipeline — lives here. Per affected thread:

1. **Do any ideas compose?** Traverse `requires:` / `unlocks:` /
   `refines:` of the new idea(s) against every existing idea. Does the
   new idea unlock a prerequisite of a shelved bet? Refine an adopted
   idea? Update `requires:` / `unlocks:` edges when new compositions
   become visible. Also surface any **orphan ideas** (ideas with 0
   referencing bets older than 30 days) whose `stages:` overlap this
   thread — these are mechanisms the wiki understands but has never
   tried to combine; propose combinations involving them. Say "nothing
   composes" only after actual enumeration.
2. **Cross-stage interactions.** Does the new component change upstream
   or downstream **assumptions** in a way that invites redesigning
   those stages too? Locally neutral swaps can be globally beneficial;
   locally beneficial swaps can be globally harmful.
3. **New combinations not proposed in any single paper.** Propose ≥1
   novel pipeline variant mixing ideas across ≥2 papers in a
   configuration none of them proposed. Write as a structured bet
   (§2.2) — assign `Bet #NNN`, fill `combines:`, `stage_target:`,
   `op_target:`, the rubric fields (`confidence`, `magnitude`, `cost`,
   `breakage_risk`), and `triggers:`. For non-drop-in scope, the bet
   body must state the topology change explicitly: nodes
   merged/split/rewired, edges appearing/disappearing, every
   neighboring node's contract re-checked. Hand-waved topology bets
   rejected. Check Shelved bets before proposing to avoid
   re-proposing known non-compositions.
4. **Cross-thread transfer.** Could this idea, or any backlog
   candidate, improve a *different* thread's pipeline? Note in both.
5. **Capability gaps refresh.** Regenerate the thread's `## Capability
   gaps` section from current shelved-bet `triggers:` and unmet
   upstream properties. This is enforced in Pass B (there is no `lint
   capability-gaps` — freshness is maintained at write-time).
6. **Verdict on the pipeline as a whole.** Should it be revised as a
   coherent whole (several stages redesigned together, not one swap)?
   If yes, propose the revised pipeline, preserve the prior via
   Pipeline lineage, record cross-stage rationale.

If Pass B produces nothing — no compositions, no novel combinations, no
cross-thread transfer — state it explicitly in the ingest report.
Silent Pass B almost always means it was skipped.

Preserve prior hypotheses with strikethrough / "superseded by" — don't
rewrite (§6.3).

#### Step 6 — Update `index.md`

Add new pages; bump `updated:` on touched ones.

#### Step 7 — Append to `log.md`

Format per §5. Include `local_paper:` path.

#### Step 8 — Report back

Files created/modified (link syntax), local paper path, contradictions
or open questions surfaced.

**Budget**: a single ingest typically touches 5–15 wiki pages. More
than 20 is a smell (creating pages for trivia). Fewer than 3 is a smell
too (missing cascade updates).

A good ingest produces:
- ≥1 concrete Pass A per-OP pipeline evaluation (even if "does not
  beat any SOTA");
- an explicit Pass B note per affected thread (even if "no new
  combinations surfaced this round");
- updated Capability gaps on every thread touched in Pass B.

A paper page without writing anything in any thread's SOTA pipelines,
Candidate components, Open questions & synthesis bets, or Capability
gaps is a smell: Step 2 or Step 5 was skipped.

### 3.2 Query

1. Read `index.md` first; then read relevant pages. Don't grep raw
   sources unless the wiki clearly doesn't cover it. If a source is
   needed and the local file is missing, do the cache check from
   §3.1 Step 1 first.
2. Answer with citations (relative links to wiki pages).
3. If the wiki is insufficient, say so: "the wiki doesn't cover X —
   want me to ingest a source on it?"
4. If the answer is substantive (comparison, synthesis, new
   connection), offer to file it: "save as `wiki/threads/<slug>.md`?"
   Good answers are wiki material, not chat fluff.

### 3.3 Thread creation (goal-first)

Any workflow that creates a new thread — ingest Step 5 proposing one,
user requesting one, `lint stage-coverage` suggesting one — **must** run
this before writing the file:

1. **Propose goal.** From the seed paper(s), draft a one-paragraph
   `## Goal` capturing what the thread is trying to do.
2. **Propose contract** (optional). Draft a `## Goal contract` YAML
   block with best-guess `metric:`, `target_regime:`, `constraints:`,
   `required_capabilities:`. Any field the seed doesn't determine,
   leave blank or comment `# to be filled`. The contract is optional
   — if the seed doesn't support a meaningful contract, omit it.
3. **Propose operating points.** Default `[op:default]`. If the seed
   clearly implies distinct regimes (speed vs. quality, object vs.
   scene, offline vs. realtime), propose up to 3 OPs with short
   qualifiers.
4. **Present to the user for edit.** Show goal + contract + OP list +
   frontmatter stub. User accepts / edits / rejects. Do not commit the
   thread file until explicit approval.
5. **Cap check**: ≥4 proposed OPs → hard fail (§6.9). Split into two
   threads.

### 3.4 Lint

Bare `lint` diagnoses (read-only). `lint <action>` fixes a specific
category with per-item approval.

#### Bare `lint` — diagnose

Scan and report (don't auto-fix):
- **Contradictions**: pages making incompatible claims.
- **Stale claims**: older pages whose claims a newer source superseded.
- **Orphans**: pages with zero inbound references. A reference is an
  Obsidian wikilink (`[[slug]]`) **or** a relative markdown link to a
  wiki page. Per §2, concept/method/thread pages are linked as
  wikilinks while paper citations use markdown links — count both.
- **Missing pages**: concepts/methods referenced ≥3 times with no page.
- **Broken links**: wikilinks to non-existent pages.
- **Frontmatter drift**: missing `updated:`, empty `sources:`, etc.
- **Stale threads**: `thread.updated <` any `source.updated` — narrative
  hasn't absorbed its own citations (remedy: `lint stale-threads`).
- **Missing papers**: `url:` set but no local file (remedy: `lint fetch`).
- **Missing code**: no `code:` and no `no code found (<date>)` body
  marker, or marker older than 6 months (remedy: `lint find-code`).
- **Missing license**: `url:` without `license_paper:`, `code:` without
  `license_code:`, dataset without `license_dataset:` (remedy: `lint
  licenses`).
- **Restrictive licenses** (informational): `license_code:` in
  {non-commercial, research-only, unknown} or `license_dataset:` in
  {non-commercial, custom-research}. Usability map, not a fix target.
- **Investigation suggestions**: 3–5 follow-up questions or source hunts.

#### `lint <action>` — targeted fixes

| Command | What it does | Writes |
|---------|--------------|--------|
| `lint fetch` | Downloads papers with `url:` but no local file (after `git clone`, after cache wipe). | `papers/` cache only |
| `lint frontmatter` | Fills missing `updated:`, empty `sources:`, drift. | `wiki/` pages |
| `lint orphans` | Proposes inbound wikilinks for orphan pages. | `wiki/` pages |
| `lint stale-threads` | Threads behind their own cited sources → targeted re-cascade. | `wiki/threads/*.md` |
| `lint find-code` | Re-searches for official/canonical code on pages without it (or with `no code found` >6 months old). Fills `code:` + `license_code:`. | `wiki/papers,methods,datasets/*.md` |
| `lint licenses` | Fills missing `license_paper:` / `license_code:` / `license_dataset:` on pages whose underlying resource exists. | `wiki/papers,methods,datasets/*.md` |
| `lint idea-duplicates` | Candidate duplicate ideas (same stage + overlapping mechanism tokens) for manual merge via `equivalent_to:` / `refines:`. | `wiki/ideas/*.md` |
| `lint unlocks-fired` | New ideas whose slugs match a shelved bet's `triggers:` → surface for re-evaluation. | read-only |
| `lint stage-coverage` | Stages referenced with no page, or stage pages with no fillers; idea-by-stage × thread matrix. | read-only |
| `lint synthesis-pressure` | Per thread: ingests vs. new Pass B bets over trailing N days. Low ratio = regressing to reading-list mode. | read-only |
| `lint type-check` | Per-node I/O compat, `co_requires:` bundle presence, **assumption-vector compatibility** against §7 registry. | read-only |
| `lint topology-drift` | Thread SOTA nodes/edges vs. cumulative Pipeline lineage — node without lineage entry, or lineage referencing no current/retired node. | read-only |
| `lint bridge-coverage` | Thread nodes with incompatible-I/O neighbors and no bridge → candidate bridge-idea stubs. | `wiki/ideas/*.md` stubs |
| `lint bets` | Sorts bets by `magnitude × confidence ÷ cost ÷ breakage_risk`; flags bets missing rubric fields; surfaces bets with `status: proposed` >30 days old. | read-only |
| `lint design-closure` | Three-way reconciliation: designs with non-pending `outcome:` and bet still `in-design`; validated bets whose combined ideas lack `validated_in:`; orphan designs (no `realizes_bet:`). | read-only |
| `lint orphan-ideas` | Ideas referenced by 0 bets older than 30 days. | read-only |

Contract: **bare `lint` is always read-only; `lint <action>` may write
but always asks first.** Enforced-in-workflow (not lints): goal contract
presence (§3.3), capability gaps freshness (§3.1 Pass B), Pareto cap
≤3 OPs (§6.9).

#### 3.4.1 Lint action procedures

All action lints share the same 5-step shape:

1. **Scan** — enumerate candidate pages per the action's selector.
2. **Filter** — reduce to the subset actually needing action.
3. **Present** — numbered table to the user, with per-page evidence.
4. **On per-item approval, apply** — make the write.
5. **Report** — counts + list of items touched + failures. Append to
   `log.md`.

Action-specific notes:

- **`lint fetch`** — use `curl -fL` (fails on HTTP errors rather than
  writing error pages to disk). Delete partial files on failure. arXiv:
  `curl -fL -o <local_paper_path> https://arxiv.org/pdf/<id>`. Papers
  without `url:` are silently skipped (listed separately so the user
  can add URLs). **Primary use case**: after a fresh `git clone`,
  repopulate the `papers/` cache.
- **`lint stale-threads`** — staleness = `thread.updated <
  max(source.updated)`. Deterministic; flag with lag in days. Do
  **not** auto-rewrite — the targeted re-cascade requires per-thread
  approval. False negative: a paper ingested but not in any thread's
  `sources:` won't be caught here — `lint orphans` covers that.
- **`lint find-code`** — same search order as ingest Step 0.6. The
  LICENSE readout is critical; unreadable license kills the candidate.
  Non-commercial is a valid result (e.g. 3DGS). Refresh the `no code
  found (<date>)` marker on misses so the 6-month cadence stays
  honored. Never overwrites an existing `code:` (stale repo links are
  a manual fix). Respect GitHub / Papers-with-Code rate limits.
- **`lint licenses`** — `unknown` is a legitimate value (ambiguous ≠
  missing). Non-SPDX values recorded verbatim with gloss. Wiki-write
  only.
- **`lint type-check`** — **assumption-vector sub-check**: compute the
  union of `assumptions:` across every adopted idea per OP; flag any
  conflicting pair from the §7 registry.
- **`lint bets`** — also surfaces bets where `op_target:` references an
  OP not declared in the thread's `operating_points:` (stale bet).
- **`lint design-closure`** — reports three categories separately
  (outcome-drift, missing-validated_in, orphan-design) so fixes are
  independent.

## 4. `index.md` format

Grouped by category. Each line:
`- [<title>](<relative-path>) — <one-line hook> · _updated YYYY-MM-DD_`

Keep human-navigable. If the index grows past ~300 lines, split into
sub-indexes (`index-papers.md`, `index-methods.md`) linked from
`index.md`.

## 5. `log.md` format

Append-only, chronological, newest at bottom. Every entry starts with
a consistent prefix so `grep "^## \[" log.md | tail -20` works:

```
## [YYYY-MM-DD] ingest | <paper short title>
- Created: wiki/papers/<key>.md, wiki/ideas/<slug>.md, wiki/stages/<slug>.md
- Updated: wiki/threads/<thread>.md (op:<name> lineage entry)
- Pipeline impact: <thread>:<op>: <stage> updated (<old> → <new>) · <thread>: Pass B — no change warranted
- Synthesis bet: Bet #NNN filed — <one-line novel combination>
- Notable: <interesting finding or open question>

## [YYYY-MM-DD] query | <short question>
- Touched: <pages read>
- Filed as: wiki/threads/<slug>.md (if saved)

## [YYYY-MM-DD] lint
- Contradictions: 2 · Orphans: 1 · Stale: 0
- Followups suggested: see entry body

## [YYYY-MM-DD] lint <action>
- <action-specific counts>
```

## 6. Hard rules

1. **Never edit source papers.** Files in `papers/` and `raw/` are
   read-only after placement (renaming/moving during ingest is allowed).
2. **Never invent citations.** Unsourceable → `[!needs-source]`.
3. **Never silently overwrite a contradicted claim.** Preserve the
   prior position (strikethrough / "superseded by") with a link to
   the new source.
4. **Stay in scope.** Photogrammetry + ML research. Off-topic → ask.
5. **Don't over-create pages.** A name-drop is not a page. A page must
   be referenced by ≥2 others or materially advance a thread.
6. **Cite down to the page.** Factual claims trace to `sources:`
   frontmatter entries.
7. **Obsidian is the read-side UI.** Wikilinks, frontmatter, relative
   paths must work without plugins beyond Dataview.
8. **No emojis in wiki content** unless the user asks.
9. **Pareto cap**: a thread has ≤3 operating points. Need a 4th → split
   the thread. Enforced at thread creation and edit.
10. **Goal first.** Every thread has a `## Goal`; `## Goal contract`
    is optional but proposed at creation and user-approved before
    commit (§3.3).
11. **Ideas are first-class.** Every novel contribution lives in
    `wiki/ideas/<slug>.md`; threads and bets reference by wikilink
    (§1.6, §2.1). No back-fill escape hatch on ingest.
12. **Type-check compositions.** Filler idea's I/O must be compatible
    with the node's stage (§2.1 stage page); upstream
    `provides_properties:` must satisfy the filler's
    `requires_upstream_property:`. Mismatches → retype or add a bridge.
13. **Topology honesty.** `multi-stage-collapse` / `stage-split` /
    `topology-rewrite` / `new-paradigm` ideas are not 1:1 stage
    fillers. The thread pipeline reflects the actual DAG change,
    recorded as a topology-change lineage entry (§2.1).
14. **Bundles travel together.** `co_requires:` ideas must all be
    present as fillers in the same pipeline — partial adoption is a
    type-check failure.
15. **Licenses never block bets or SOTA choices.** License info is informational — tracked in frontmatter and in [License Audit](wiki/meta/license-audit.md) for implementation-time planning. Bet adoption, `confidence`, `status`, and SOTA-pipeline composition are decided on mechanism merit alone.

## 7. Assumption conflict registry

Closed-world list of assumption pairs that cannot co-exist in one
pipeline. Used by `lint type-check`'s assumption-vector sub-check.
Editable as new conflicts emerge — append a row and bump `updated:`
on any thread whose SOTA pipeline then fails the check.

| Left | Right |
|------|-------|
| `static-scene` | `dynamic-objects` |
| `bounded-scene` | `unbounded-scene` |
| `posed-input` | `unposed-input` |
| `metric-scale` | `up-to-scale` |
| `known-intrinsics` | `uncalibrated-input` |
| `calibrated-photometric` | `raw-exposure-unknown` |
| `realtime-inference` | `per-scene-optimization-required` |
| `surface-primitive` | `volumetric-primitive` |
| `single-camera` | `multi-rig-synchronized` |

## 8. Co-evolution

This file is not frozen. When the user develops a workflow that works
(or catches one that doesn't), update the schema. Log schema changes
under a `## [YYYY-MM-DD] schema-change` entry in `log.md`.

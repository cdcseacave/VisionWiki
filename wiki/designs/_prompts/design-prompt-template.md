---
title: Design-Prompt Template (Research-Synthesis Variant)
type: reference
tags: [meta, prompt, design-process]
created: 2026-04-15
updated: 2026-04-15
---

A reusable scaffolding for prompts that ask the LLM to produce a `wiki/designs/`
page. Use when the goal is a buildable blueprint that rests on the wiki's
accumulated synthesis, not a one-paper summary. Copy, fill in the `<< ... >>`
slots, delete sections that don't apply.

The template was distilled from the retrospective on the
[language-grounded-3dgs](../language-grounded-3dgs.md) prompt — which
underspecified what "research and think deeply" actually requires.

---

## Template

> Design the best pipeline to << one-sentence capability statement —
> what the system must do, in the user's own words >>. Concrete interaction
> examples the system must support: << 2–4 examples like "select all chairs
> in the scene", "delete all lamps in this room", "hide the red mug" >>.
>
> Treat this as a **research synthesis task, not a paper summary**. Read
> across the relevant threads — for this problem that is [[<< primary
> thread >>]] and [[<< secondary thread >>]] — and reason about which
> foundation model / module supplies each distinct signal the design needs
> (<< enumerate the signals: e.g. semantics, spatial coherence, mask
> boundaries, cross-view instance identity, geometric grounding >>).
> Evaluate whether the prior-generation stack should be replaced by its
> current-generation equivalents (<< name the specific upgrades: e.g. SAM-1
> → SAM 3 native IDs, CLIP + DINO + SAM composition → RADIOv2.5 unified
> backbone, two-stage → one-stage training >>).
>
> Think deeply about the design space along these axes — address each
> explicitly, do not collapse them:
>
> 1. **<< axis 1: what is stored / represented >>** — the storage-shape
>    decision, its memory cost, its dimensionality tradeoff.
> 2. **<< axis 2: how cross-instance/cross-view consistency is obtained >>**
>    — what supplies the consistency signal, and consequences for upstream
>    or downstream stages.
> 3. **<< axis 3: how the primary supervision is constructed >>** — which
>    teacher, which loss, which regularizer.
> 4. **<< axis 4: how queries / inference execute >>** — what is precomputed
>    at train time vs. at query time, with a per-query latency budget.
> 5. **<< axis 5: where the dominant method breaks, and the fallback >>** —
>    the failure modes of the primary signal and the escalation path.
>
> ### Numeric success criteria (non-negotiable)
>
> - Accuracy: << metric + dataset + target, e.g. "3D-OVS mIoU ≥ LangSplat on
>   the 3D-OVS benchmark" >>.
> - Latency: << per-query target, e.g. "text-to-selection < 100 ms on 1 M
>   Gaussians on a single consumer GPU" >>.
> - Training cost: << relative bound, e.g. "≤ 1.5× vanilla 3DGS on the same
>   scene" >>.
> - Memory: << per-primitive or total, e.g. "≤ 100 MB feature overhead at
>   1 M Gaussians" >>.
>
> "Reasonable" / "fast" / "accurate" without a number is not an answer.
>
> ### Required output structure
>
> - **Goal & success criteria** — numeric targets.
> - **Pipeline** — numbered stages, each with: *(source paper + mechanism +
>   what it replaces from the prior SOTA)*.
> - **Query / interaction layer** — for each query type (e.g. text, click,
>   compositional): the compute path, data structures precomputed at train
>   time, per-query latency budget.
> - **Editing / action operations** — each user-visible action mapped to a
>   concrete operation on the underlying representation.
> - **Densification / pruning / update hygiene** — how per-primitive fields
>   stay consistent under geometry ops (splits, merges, prunes). Ad-hoc
>   handling is a smell.
> - **Performance budget** — training time, per-query latency, per-primitive
>   memory overhead, with rough FLOP / bandwidth accounting.
> - **Open questions & risks** — specific to *this* design, not generic. If
>   every risk applies to any 3D method, you've punted.
> - **Deliberate non-goals** — capabilities the design intentionally does
>   not address, and why (usually: different tradeoff, different paper).
> - **Build sequence** — ordered steps, each with one **falsifiable
>   checkpoint**; do not advance to the next step without passing it.
>
> ### Rigor gates (reject output that fails these)
>
> - **Mechanism-level, not slogan-level.** "Distill CLIP into Gaussians" is
>   not an answer; *what is rasterized, what loss compares it to what target,
>   at which hierarchy level* is.
> - **Every "X improves Y" claim cites evidence** — either an ablation in
>   the source paper or a convergent finding in an adjacent paper in the
>   wiki.
> - **Every speed claim decomposes into operations** with rough FLOP /
>   memory accounting, not "it's fast because it's precomputed."
> - **Contradictions in the literature are acknowledged, not papered over.**
>   If one paper claims A and another claims ¬A (per-scene vs. generalizable,
>   3DGS vs. voxel substrate, CLIP-only vs. dual-distillation), state the
>   design's position and why.
> - **Architectural decisions trace to the wiki.** Every stage in the
>   pipeline should cite a paper page (or flag the gap as a stub to create).
>   A design that invents its way around the wiki's synthesis is not
>   building on the wiki.

---

## Slot-filling guide

### `<< primary thread >>` / `<< secondary thread >>`

Pick from `wiki/threads/`. The primary thread is usually the one whose
synthesis bets the design is operationalizing. The secondary thread is the
one it inherits conceptual grounding from (e.g. a 3D-lifting design inherits
from the corresponding 2D composition thread).

### The five axes

These are problem-dependent but should always be **orthogonal**: each axis
should represent a decision that can be made independently of the others.
Common axis templates:

- **What is stored** (fused vs separated; raw vs compressed; continuous vs
  quantized).
- **How consistency is obtained** (association loop vs native IDs; explicit
  reg vs emergent from joint training).
- **How supervision is constructed** (which teacher, which loss, which
  hierarchy).
- **How inference executes** (per-query re-encode vs precomputed indices;
  hash vs k-NN vs brute force).
- **Where the primary signal breaks and what fallback covers it** (is there
  a VLM / LLM fallback, a specialist model, a human-in-the-loop step?).

### Numeric targets

If you do not know the right number yet, write "to be determined from the
X benchmark" — but do not omit the bullet. The number's *presence* forces
the design to propose a measurable checkpoint, not hand-wave.

### Novel-combinations clause

This is the single most important clause. Without it the LLM will reliably
pick one paper, dress it up with three citations, and call it a design.
Concrete enforcement phrasings that work:

- "at least one idea-pairing across ≥ 2 papers that no single paper
  proposed"
- "the design must cite a synthesis bet from [[<< thread >>]] and either
  operationalize it or argue why it's wrong"
- "if you find yourself re-implementing paper X, stop and justify why
  nothing from the thread's Candidate Components section should combine
  with X"

### Deliberate non-goals

Naming what the design does *not* solve is a rigor signal. It forces the
designer to know the shape of the surrounding problem space, not just the
slice they're claiming. Good non-goals are usually *different tradeoffs*,
not *different problems*: "this design doesn't address zero-shot cross-scene
generalization" is a non-goal (CLIP-GS's lane); "this design doesn't solve
fluid simulation" is noise.

## Anti-patterns to reject in the output

- **Summary dressed as design.** Section headers present but every bullet
  is paraphrased from one source paper.
- **Numeric targets rounded to vibes.** "~fast", "~accurate", "small
  memory" instead of numbers.
- **Papered-over contradictions.** E.g. using LangSplat's per-scene
  autoencoder *and* claiming zero-shot cross-scene generalization without
  addressing the contradiction.
- **Generic risks.** "May not scale" / "hyperparameters matter" — every 3D
  method has these; they are not risks specific to *this* design.
- **Build sequence without falsifiable checkpoints.** "Step 3: train the
  model" is not a checkpoint; "Step 3: 3D-OVS mIoU on held-out views
  matches LangSplat within 1%" is.

## How to use

1. Copy the template block into your prompt.
2. Fill the slots for the specific design.
3. Paste into a fresh Claude session (the `raw/` → ingest → `wiki/` context
   loads automatically via `CLAUDE.md`).
4. When the output arrives, run each section past the rigor gates — treat
   the gates as a checklist, not a suggestion.
5. If the design violates a gate, respond with *"violates gate N — redo
   section X with mechanism-level detail"* rather than accepting the output
   and repairing it yourself. The repair should live in the design, not
   in your head.

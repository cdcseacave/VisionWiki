---
title: Compact trajectory memory — 6-token-per-frame eviction for streaming SfM
type: idea
source_paper: wiki/papers/chen2026_lingbot-map.md
also_in: []

scope: stage-swap
stages: [feed-forward-sfm.long-context-memory]

inputs: [per-frame-token-set-including-camera-anchor-register-and-image-tokens]
outputs: [compact-per-frame-summary-6-tokens, image-tokens-discarded]
assumptions: [vit-backbone-produces-register-tokens, camera-and-anchor-tokens-accumulate-geometric-summary-via-attention, streaming-or-chunked-inference]
requires_upstream_property: [per-frame-camera-token, per-frame-anchor-token, per-frame-register-tokens-4]
requires_downstream_property: [attention-mechanism-can-attend-to-sparse-per-frame-summaries]
learned_params: [register-token-embeddings, anchor-token-embedding, attention-layers-pre-eviction]
failure_modes: [very-long-sequences-beyond-10k-frames-lose-fine-detail, 6-tokens-insufficient-for-some-scene-classes-e-g-repetitive-texture, no-re-hydration-path-if-a-frame-becomes-relevant-again]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [long-context-memory, streaming, eviction, compression, kv-cache, sfm, feed-forward]
created: 2026-04-24
updated: 2026-04-24
status: unclaimed
---

## Mechanism

Streaming attention over a video stream suffers from KV-cache growth: each new frame contributes $M$ image tokens ($M \approx 500$ for 518×378 with patch=14 DINOv2 ViT), and causal attention retains all of them. At $T=10,000$ frames, that's $5 \times 10^6$ tokens per attention computation — intractable.

The compact-trajectory-memory mechanism:

1. At the tokenization stage, each frame is augmented with a **camera token** $\mathbf{c}$, four **register tokens** $\mathbf{r}_j$, and a learnable **anchor token** $\mathbf{a}$ (6 tokens total, plus $M$ image tokens). Register tokens follow the Vision-Transformers-Need-Registers convention (Darcet 2024): dedicated learnable tokens that absorb attention sinks and accumulate summary features.
2. In each alternating Frame-Attention + Cross-Frame-Attention block, the 6 per-frame "summary" tokens (camera + anchor + 4 register) accumulate a learned abstraction of the frame's geometric content via in-block attention over image tokens.
3. For frames outside the anchor set + local pose-reference window, the attention pipeline **retains only the 6 summary tokens per frame** and discards the $M$ image tokens. KV-cache entries for image tokens are evicted (and, with [[paged-kv-cache-streaming-sfm_chen2026]], the evicted pages are freed without realloc).
4. During subsequent frames' attention, the retained 6-token summaries behave as a compressed memory: camera-token ≈ frame's pose summary; register-tokens ≈ learned geometric abstractions. Requires [[video-rope-on-trajectory-tokens_chen2026]] for temporal order — without it, the compressed memory is permutation-symmetric and largely useless (see that idea's ablation).

Per-frame attention-memory growth: 6 tokens/frame instead of $M+6$. Analytic reduction: $\approx M/6 \approx 80×$ for $M=500$. At $T=10^4$, causal attention retains $\approx 5 \times 10^6$ tokens; this mechanism retains $\approx 6 \times 10^4$.

## Why it wins

Ablation (Table 6 row 2 → 4): adding context tokens on top of anchor init improves AUC@3 from 13.63 → 15.75 (+2.12) and ATE from 7.88 → 7.46 (−0.42). Modest alone — the memory's value is only fully realized when paired with Video RoPE (row 4 → 5: ATE 7.46 → 5.98, largest single mover).

Table 7 additionally shows that **truncating distant image tokens does not hurt accuracy** — in fact, bounded window (k=64) + trajectory-memory beats full causal attention on ATE (5.98 vs 6.60) and RPE-trans (1.33 vs 1.50), with 1.7× higher FPS and 2.7× lower memory. Paper's interpretation: distant image tokens add noise more than signal; compact summaries preserve the geometric signal while filtering noise.

Oxford-Spires dense (3840 frames): LingBot maintains ATE 7.11 while CUT3R (which uses aggressive RNN compression rather than compact-token eviction) degrades to 32.47. Evidence that this specific compression scheme — 6 independent per-frame tokens with temporal encoding — beats a single shared RNN state.

## Preconditions & compatibility

- Requires a tokenizer that produces per-frame camera + anchor + register tokens. Any VGGT-family or DINOv2-backboned model has these patterns.
- Requires a downstream attention mechanism that can attend to sparse summary tokens. GCA satisfies this; standard causal attention would also work (with eviction policy).
- Requires temporal positional encoding for the summaries (see [[video-rope-on-trajectory-tokens_chen2026]]) — otherwise the compressed memory is permutation-symmetric and largely ineffective.
- Compatible with: any feed-forward SfM with KV-caching. The eviction policy is architecture-independent.
- Does NOT co-require the GCA three-context topology — this mechanism is the *memory slot*, and can plug into any streaming-SfM model that needs a bounded long-range memory.

## Trade-offs vs. alternatives

- **vs. full causal attention**: $O(1)$ vs $O(T)$ per-frame growth, plus accuracy improvement at long sequences. Clear win.
- **vs. CUT3R RNN state**: compact-trajectory-memory has *per-frame* independence (each frame's summary is its own 6 tokens), while RNN compresses everything into one shared state. Empirically CUT3R forgets on long sequences; this mechanism doesn't. Cost: 6T tokens vs O(1) for RNN — trading slightly-growing memory for much better retention.
- **vs. TTT fast-weights (TTT3R, ZipMap)**: TTT updates parameters at inference to encode past frames into the model itself; this mechanism encodes past frames as explicit tokens. Trade-off: TTT is more parameter-efficient at inference but requires gradient computation; compact-memory is fully feed-forward but grows linearly in $T$ (at 6 tokens/frame).
- **vs. FastVGGT token merging**: FastVGGT retains $M$ tokens/frame but merges them aggressively within a frame; compact-memory retains 6 tokens/frame across ALL past frames. Orthogonal compression axes — theoretically stackable.

## Portability

The mechanism is separable from the rest of GCA and portable to other streaming architectures:

- Plug-in replacement for CUT3R's RNN state: replace the recurrent update with per-frame 6-token summaries; test whether long-sequence retention improves. Proposed as Bet #033 on the thread.
- Plug-in for StreamVGGT's full-history KV-cache: replace evicted-frame retention with 6-token summaries; test whether memory bounds can be imposed without accuracy loss.
- Composable with TTT mechanisms: TTT updates could operate on the compact-memory tokens directly. Proposed as Bet #032.

## Open questions

- What do the 4 register tokens encode geometrically? No register-count ablation in the paper.
- Is 6 tokens/frame the right number? No sweep.
- Does re-hydrating (re-attending to the full image tokens of a past frame) help when a revisit is detected?
- Could compact-memory be adaptive — retain more tokens for "important" frames? Paper uses uniform 6-token eviction.
- How does eviction interact with loop closure (LingBot's stated limitation)?

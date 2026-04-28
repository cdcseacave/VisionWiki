---
title: Paged KV-cache for streaming feed-forward SfM inference
type: idea
source_paper: wiki/papers/chen2026_lingbot-map.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.long-context-memory]

inputs: [streaming-attention-kv-cache-with-frequent-evict-and-append]
outputs: [same-attention-outputs-with-reduced-memory-management-overhead]
assumptions: [attention-layer-supports-paged-kv-cache, flashinfer-or-equivalent-paged-attention-kernel-available]
requires_upstream_property: [kv-cache-layout-is-mutable]
requires_downstream_property: [attention-kernel-supports-page-table]
learned_params: [none-systems-level-optimization]
failure_modes: [small-memory-savings-on-short-sequences, page-fragmentation-under-pathological-eviction-patterns, kernel-compatibility-limits-hardware-portability]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [systems, inference-optimization, kv-cache, streaming, paged-attention]
created: 2026-04-24
updated: 2026-04-24
status: unclaimed
---

## Mechanism

Streaming attention with explicit frame eviction (see [[compact-trajectory-memory-tokens_chen2026]]) generates a pattern of KV-cache updates that stresses contiguous memory layouts: every new frame appends entries and evicts old ones, triggering either periodic full-cache reallocation or expensive compaction. This overhead compounds per frame and dominates the inference-time budget once it becomes a significant fraction of total attention cost.

**Paged KV-cache** (borrowed from PagedAttention in LLM serving [Kwon 2023]) addresses this by storing the KV-cache in non-contiguous fixed-size pages rather than a single contiguous tensor. A page table maps logical positions (sequence indices) to physical memory pages. Eviction releases a page back to the pool; appending allocates a new page. No data movement is required on either operation.

LingBot implements this via the FlashInfer runtime [ref 95], which natively supports paged KV-cache layouts and provides optimized attention kernels that work directly over the page table.

## Why it wins

From §3.4: at 518×378 input resolution, sequences up to 1000 frames, and a sliding window of 64 frames, FlashInfer with paged KV-cache achieves **~20 FPS**, vs. **~10.5 FPS** for an identical PyTorch implementation with a contiguous KV-cache that uses the same attention kernel logic but with naive re-alloc. ≈2× speedup.

The speedup is purely systems-level — no reconstruction-quality change. Paper does not include a quality-ablation row for this (and wouldn't expect one; it's a representational change to the cache, not a model change).

This is a concrete *measured* gain (not analytical), which makes it stronger evidence than most systems-level papers provide.

## Preconditions & compatibility

- Requires an attention runtime/kernel that supports paged KV-cache natively (FlashInfer, vLLM-style page tables, or similar). Hardware implementations: NVIDIA CUDA + compatible kernels; pure-PyTorch fallback is what LingBot's baseline measures (10.5 FPS).
- Compatible with: any streaming feed-forward SfM that uses KV-caching with eviction/append patterns. This includes LingBot GCA, CUT3R (with RNN-to-token substitution), TTT3R, Stream3R, StreamVGGT — all of these stream inference and all could benefit.
- Does NOT require GCA's three-context decomposition. It's a pure inference-runtime optimization orthogonal to the model architecture.

## Trade-offs vs. alternatives

- **vs. contiguous KV-cache with periodic realloc**: ~2× speedup measured; memory overhead is a small page-table structure (negligible). Clear win.
- **vs. contiguous cache + ring-buffer**: ring-buffers avoid realloc for bounded windows but can't efficiently handle variable-size eviction (e.g., LingBot's keyframe-conditional eviction). Paged cache handles both.
- **vs. TTT-family KV-cache-free approaches**: TTT replaces the cache with fast-weights, eliminating the cache-management problem entirely. Trade-off: TTT has its own compute cost; paged-KV-cache preserves exact-attention semantics.
- **Cost**: implementation complexity (page-table + compatible kernel). Not free to engineer, but FlashInfer provides a drop-in path.

## Portability

Highly portable systems-level optimization. Any streaming SfM with KV-caching and eviction benefits. Candidate transfers to:

- [[shen2025_fastvggt|FastVGGT]] — already compresses tokens via merging; paged cache is orthogonal and stackable.
- [[feng2025_quantvggt|QuantVGGT]] — W4A4 compresses KV entries bit-wise; paged layout is orthogonal, stackable.
- [[chen2026_ttt3r|TTT3R]] — TTT3R doesn't use a traditional KV-cache (uses recurrent fast-weights), so less applicable.
- Traditional CUT3R-family models that maintain explicit history — direct drop-in.

Paged KV-cache is one axis of a multi-axis inference-optimization stack for streaming SfM (token-count, attention-sparsity, numerical-precision, cache-layout).

## Open questions

- Does FlashInfer's paged-attention kernel support all variants of the GCA three-context mask, or are some patterns awkward? Paper doesn't specify.
- How does page fragmentation behave under long-running inference (tens of thousands of frames)?
- Is there a case where page-table overhead exceeds realloc savings? Probably only at very short sequences where neither mechanism matters.
- Can the page-size be tuned to match the 6-token-per-frame evicted summaries (from [[compact-trajectory-memory-tokens_chen2026]])? Potential co-optimization.

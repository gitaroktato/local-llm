# AGENTS.md

Personal lab repo for running LLMs locally on custom hardware.
Repo: github.com/gitaroktato/local-llm

## Instructions

Any fundamental concept learnt from this project should be saved inside the root `AGENTS.md` file.

- Plan work ends with updating/creating the GitHub issue — do NOT execute the plan's steps inline; the issue is the deliverable, execution happens later from it.

## Runtime Concepts (verified on llama.cpp b12246)

- Qwen3.8-27B (`qwen35` arch) is hybrid attention: 65 blocks = 17 full-attention layers (kv heads=4, head_dim=256) + 48 SSM/linear-attention layers; native context 262,144. KV cache grows only on the full-attention layers: ~68 KiB/token at f16, ~36 KiB at q8_0.
- Quantized KV works WITH `--flash-attn on` in this build (q8_0/q5_1/q4_0 paths in `ggml/src/ggml-cuda/fattn.cu`; CLI accepts f32/f16/bf16/q8_0/q4_0/q4_1/iq4_nl/q5_0/q5_1). Older builds/Studio UIs rejecting low KV quants is outdated.
- `--cache-ram` is a server prompt cache for idle slots (cold-start mitigation, issue #35), NOT active-KV offload — it does not extend the context window.
- VRAM budget must be done per GPU, not total: tensor split is even, but the vision subcontext pins entirely to CUDA0 (mmproj model ~0.89 GB F16 / ~0.44 GB Q8_0 + warmup compute buffer ~0.25 GB) — GPU0 is the binding device. At 192k/q8_0+MTP idle: GPU0 free 471 MiB vs GPU1 free 1799 MiB (issue #48).
- MTP spec decoding has hidden VRAM costs: the draft context holds its own KV cache for 1 layer (the final nextn block; main KV covers the other 16 attention layers) at f16 by default (~0.75 GB at 192k), and the RS buffer grows to 4× base SSM state (`n_rs_seq=3`, 299 vs 75 MiB). DO NOT use `--spec-draft-type-k/v q8_0` on b12246: it grows the spec-context compute buffer 527→1212 MiB (larger FA workspace for quantized draft KV) vs only ~164–257 MiB saved in draft KV — net VRAM loss, no acceptance-rate benefit. Keep draft KV f16.
- The compute/graph pool grows on demand during prompt processing (warmup only captures small graphs); a config where only static allocations fit at startup will OOM on the first large prompt. Keep ≥ ~0.5 GB idle free per GPU; `--ubatch-size` scales the prompt-eval graph pool (1024→512 saves ~300–500 MiB/GPU, prefill-only cost).

## Session Logs

`scripts/export-sessions.sh` exports all opencode sessions of this project to `.sessions/<sessionID>.json` (gitignored). Re-runs skip up-to-date sessions and only re-export ones changed since their file's mtime; `--force` re-exports everything.

## Privileged Commands (sudo)

The owner executes ALL `sudo` commands personally. The agent must NEVER run sudo itself (no passwordless probing either). Instead: prepare everything else locally, then hand the exact sudo commands to the owner as a copy-paste block and wait for confirmation before continuing.

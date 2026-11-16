# llama-cpp Configs


Tip: Add `--metrics` for metrics


## Qwen 3.6 Q4_XL with 128k context window using Q5_1


```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/Qwen3.6-27B-UD-Q4_K_XL.gguf --port 35019 -c 128000 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads -1 --jinja --cache-type-k q5_1 --cache-type-v q5_1 --spec-default --chat-template-kwargs {"enable_thinking": true} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/mmproj-F16.gguf
```

### Split-mode - tensor (simple run)

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/Qwen3.6-27B-UD-Q4_K_XL.gguf --metrics --port 8888 -c 128000 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads 16 --split-mode tensor --fit off --jinja --cache-type-k bf16 --cache-type-v bf16 --spec-default --chat-template-kwargs '{"enable_thinking": true}' --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/mmproj-F16.gguf
```


## Qwen 3.6 35B-A3B Q4_XL with 128k context window

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-35B-A3B-GGUF/snapshots/a483e9e6cbd595906af30beda3187c2663a1118c/Qwen3.6-35B-A3B-UD-Q4_K_XL.gguf --port 48713 -c 262144 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads -1 --jinja --spec-default --chat-template-kwargs {"enable_thinking": false} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-35B-A3B-GGUF/snapshots/a483e9e6cbd595906af30beda3187c2663a1118c/mmproj-F16.gguf
```

### Split-mode - tensor (simple run)

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-35B-A3B-GGUF/snapshots/a483e9e6cbd595906af30beda3187c2663a1118c/Qwen3.6-35B-A3B-UD-Q4_K_XL.gguf  --port 8888 -c 128000 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads 16 --split-mode tensor --fit off --jinja --cache-type-k bf16 --cache-type-v bf16 --spec-default --chat-template-kwargs '{"enable_thinking": true}' --mmproj  /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-35B-A3B-GGUF/snapshots/a483e9e6cbd595906af30beda3187c2663a1118c/mmproj-F16.gguf
```


## Qwen 3.8 - validated default (build 11734, MTP n-max 2)

Measured on build 11734 (`216986d87`), dual RTX 5060 Ti tensor split: **67–72 tok/s** with MTP acceptance 0.81–0.88 (mean len 2.62–2.76). Base decode without spec is 33.8 tok/s; ngram-mod alone ~32–34 (no gain); MTP+ngram ≈ MTP-only (65.3 on a long run, ngram fired 4/3169 calls). Always set `--spec-type` explicitly — Studio's "auto" resolves to no spec implementation. See issue #41.

Build 12246 (`7a901e9a5`) note: its `--spec-draft-n-max` default is 3 (previously 2). With just `--spec-type draft-mtp` (no n-max flag) measured **74–79 tok/s**, acc rate/pos ≈ (0.80, 0.71, 0.59), mean len ~3.1 — better than the n_max=2 config above.

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/4ca720788d1e01f1bff70c033e0d0028fd02e502/Qwen3.8-27B-UD-Q4_K_XL.gguf --port 8888 --parallel 1 --flash-attn on --no-context-shift -c 128000 --ubatch-size 1024 --alias unsloth/Qwen3.8-27B-GGUF -ngl -1 --fit off --metrics --slot-save-path /home/gitaroktato/.unsloth/studio/cache/llama-slots --jinja --split-mode tensor --spec-type draft-mtp --spec-draft-n-max 2 --chat-template-kwargs '{"enable_thinking": true, "preserve_thinking": true}' --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/4ca720788d1e01f1bff70c033e0d0028fd02e502/mmproj-F16.gguf --threads 10
```

## Qwen 3.8 - 200k context, q8_0 KV + MTP (build b12246, current)

Current stable config (issue #48): `-c 200000` with q8_0 main KV (~36 KiB/token vs ~68 f16), MTP draft with **f16 draft KV** (`--spec-draft-n-max 3`, b12246 default), `--ubatch-size 512`. Measured: ~56 ms/step constant, 50–58 t/s at 80–93k ctx in use, acceptance 0.59–0.75 (prose ~0.75, code/tool-output ~0.59), 0 graph reallocs.

Do NOT add `--spec-draft-type-k/v q8_0`: on b12246 it grows the spec-context compute buffer 527 → 1212 MiB (larger FA workspace) vs only ~164–257 MiB saved in draft KV — net VRAM loss, no perf/acceptance benefit. Keep draft KV f16.

Full 262k does not fit yet: needs mmproj Q8_0 (`Qwen3.5-27B-mmproj-Q8_0.gguf` frees ~443 MiB on GPU0) and possibly `--spec-draft-n-max 2` or ubatch 256 (issue #48).

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/4ca720788d1e01f1bff70c033e0d0028fd02e502/Qwen3.8-27B-UD-Q4_K_XL.gguf --port 39967 --parallel 1 --flash-attn on --no-context-shift -c 200000 --ubatch-size 512 --video-fps 1 --video-timestamp-interval 2000 --alias unsloth/Qwen3.8-27B-GGUF -ngl -1 --fit off --metrics --slot-save-path /home/gitaroktato/.unsloth/studio/cache/llama-slots --jinja --verbosity 4 --cache-type-k q8_0 --cache-type-v q8_0 --split-mode tensor --reasoning on --chat-template-kwargs {"preserve_thinking": true} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/4ca720788d1e01f1bff70c033e0d0028fd02e502/mmproj-F16.gguf --ctx-checkpoints 10 --threads 10 --chat-template-file /home/gitaroktato/Projects/local-llm/chat-templates/qwen3.8-froggeric-v22.3.jinja --reasoning-format deepseek --spec-type draft-mtp --spec-draft-n-max 3
```

## Qwen 3.8 - split-mode tensor (simple run)

Stale: `--spec-type ngram-mod` measured ~33 t/s, no gain over base — superseded by the MTP default above.

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/f1bfb127c64f7072bdd2cad55f258b9c8b2910fe/Qwen3.8-27B-UD-Q5_K_XL.gguf --port 8888 --parallel 4 --flash-attn on --no-context-shift -c 128000 --alias unsloth/Qwen3.8-27B-GGUF -ngl -1 --threads 16 --split-mode tensor --fit off --metrics --slot-save-path /home/gitaroktato/.unsloth/studio/cache/llama-slots --cache-type-k bf16 --cache-type-v bf16 --kv-unified --jinja --spec-type ngram-mod --spec-ngram-mod-n-match 24 --spec-ngram-mod-n-min 48 --spec-ngram-mod-n-max 64 --chat-template-kwargs '{"enable_thinking": true, "preserve_thinking": false, "reasoning_effort":"medium"}' --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/f1bfb127c64f7072bdd2cad55f258b9c8b2910fe/mmproj-F16.gguf
```

From `unsloth studio`

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/f1bfb127c64f7072bdd2cad55f258b9c8b2910fe/Qwen3.8-27B-UD-Q5_K_XL.gguf --port 37741 --parallel 1 --flash-attn on --no-context-shift -c 104448 --alias unsloth/Qwen3.8-27B-GGUF -ngl -1 --fit off --metrics --slot-save-path /home/gitaroktato/.unsloth/studio/cache/llama-slots --jinja --split-mode tensor --spec-type draft-mtp --spec-draft-n-max 2 --chat-template-kwargs {"enable_thinking": true, "preserve_thinking": true} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/blobs/cbb841a9ee0636b2ec172f5bb8df2ea8dfeb01e90fe7c6126581d662a0b4e43e
```

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/f1bfb127c64f7072bdd2cad55f258b9c8b2910fe/Qwen3.8-27B-UD-Q5_K_XL.gguf --port 43775 --parallel 1 --flash-attn on --no-context-shift -c 101632 --alias unsloth/Qwen3.8-27B-GGUF -ngl -1 --fit off --metrics --slot-save-path /home/gitaroktato/.unsloth/studio/cache/llama-slots --jinja --split-mode tensor --spec-type draft-mtp --spec-draft-n-max 2 --chat-template-kwargs {"enable_thinking": true, "preserve_thinking": true} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/blobs/cbb841a9ee0636b2ec172f5bb8df2ea8dfeb01e90fe7c6126581d662a0b4e43e --threads 16
```

Optimization run 3

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/f1bfb127c64f7072bdd2cad55f258b9c8b2910fe/Qwen3.8-27B-UD-Q5_K_XL.gguf --port 41207 --parallel 1 --flash-attn on --no-context-shift -c 79616 --ubatch-size 1024 --alias unsloth/Qwen3.8-27B-GGUF -ngl -1 --fit off --metrics --slot-save-path /home/gitaroktato/.unsloth/studio/cache/llama-slots --jinja --split-mode tensor --spec-type draft-mtp --spec-draft-n-max 2 --chat-template-kwargs {"enable_thinking": true, "preserve_thinking": true} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/blobs/cbb841a9ee0636b2ec172f5bb8df2ea8dfeb01e90fe7c6126581d662a0b4e43e --threads 10
```

Optimization run 4 (vision on CPU) - not good, eats a lot of CPU

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/f1bfb127c64f7072bdd2cad55f258b9c8b2910fe/Qwen3.8-27B-UD-Q5_K_XL.gguf --port 48705 --parallel 1 --flash-attn on --no-context-shift -c 128000 --ubatch-size 1024 --alias unsloth/Qwen3.8-27B-GGUF --fit on --metrics --slot-save-path /home/gitaroktato/.unsloth/studio/cache/llama-slots --fit-ctx 128000 --fit-target 547 --jinja --spec-type none --chat-template-kwargs {"enable_thinking": true, "preserve_thinking": true} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/blobs/cbb841a9ee0636b2ec172f5bb8df2ea8dfeb01e90fe7c6126581d662a0b4e43e --threads 10 --split-mode layer --no-mmproj-offload
```


### MTP+Ngram combined spec (final performance check, build 11734)

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/4ca720788d1e01f1bff70c033e0d0028fd02e502/Qwen3.8-27B-UD-Q4_K_XL.gguf --port 50411 --parallel 1 --flash-attn on --no-context-shift -c 128000 --ubatch-size 1024 --alias unsloth/Qwen3.8-27B-GGUF -ngl -1 --fit off --metrics --slot-save-path /home/gitaroktato/.unsloth/studio/cache/llama-slots --jinja --split-mode tensor --spec-type ngram-mod,draft-mtp --spec-draft-n-max 2 --spec-ngram-mod-n-match 24 --spec-ngram-mod-n-min 48 --spec-ngram-mod-n-max 64 --reasoning on --chat-template-kwargs {"preserve_thinking": true} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/4ca720788d1e01f1bff70c033e0d0028fd02e502/mmproj-F16.gguf --ctx-checkpoints 10 --threads 10 --chat-template-file /home/gitaroktato/Projects/local-llm/chat-templates/qwen3.8-froggeric-v22.3.jinja --reasoning-format deepseek
```

Measured (build 11734, `216986d87`, Q4_K_XL snapshot `4ca72078`): long 3973-token generation at **65.3 tok/s** overall (rolling 66–70 t/s). MTP does the work: mean acc len ~2.2–2.4, acc rate/pos ≈ (0.72, 0.50). ngram-mod fired 4/3169 calls — negligible for prose/chat; only pays off on repetitive text. ≈ same as MTP-only (~63 tok/s), still below the Oct 9 baseline of ~76–77 tok/s (see issue #41).

## Interesting things to try

- <https://www.reddit.com/r/LocalLLM/comments/1was9n0/running_qwen_38_27b_at_q4_on_16gb_vram_at_200k/>

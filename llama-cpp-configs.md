# llama-cpp Configs


## Qwen 3.6 Q4_XL with 128k context window using Q5_1


```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/Qwen3.6-27B-UD-Q4_K_XL.gguf --port 35019 -c 128000 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads -1 --jinja --cache-type-k q5_1 --cache-type-v q5_1 --spec-default --chat-template-kwargs {"enable_thinking": true} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/mmproj-F16.gguf
```

### Split-mode - tensor (simple run)

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/Qwen3.6-27B-UD-Q4_K_XL.gguf --port 8888 -c 128000 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads -4 --split-mode tensor --jinja --cache-type-k bf16 --cache-type-v bf16 --spec-default --chat-template-kwargs '{"enable_thinking": true}' --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/mmproj-F16.gguf
```

### Split-mode - tensor (2x RTX 5060 Ti 16GB, 32GB total)

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/Qwen3.6-27B-UD-Q4_K_XL.gguf --port 8888 -c 128000 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads 16 --split-mode tensor --jinja --cache-type-k q8_0 --cache-type-v q8_0 --spec-default --chat-template-kwargs '{"enable_thinking": true}' --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/mmproj-F16.gguf
```

#### Key changes from bf16 config:
- `--cache-type-k/v q8_0` — ~50-70% less KV cache memory, fits within 2x16GB VRAM
- `--threads 8` — use physical core count (run `nproc` to verify), not negative value

#### If VRAM still tight, reduce context:
```bash
# 64k context — more headroom, still sufficient for most coding sessions
-c 65536
```

#### If you can afford quality loss for max context:
```bash
# q4_0 cache — smallest footprint, but more quality degradation on long contexts
--cache-type-k q4_0 --cache-type-v q4_0
```

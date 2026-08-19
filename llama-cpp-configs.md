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


## Qwen 3.8 - split-mode tensor (simple run)

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/f1bfb127c64f7072bdd2cad55f258b9c8b2910fe/Qwen3.8-27B-UD-Q5_K_XL.gguf --port 8888 --parallel 4 --flash-attn on --no-context-shift -c 128000 --alias unsloth/Qwen3.8-27B-GGUF -ngl -1 --threads 16 --split-mode tensor --fit off --metrics --slot-save-path /home/gitaroktato/.unsloth/studio/cache/llama-slots --cache-type-k bf16 --cache-type-v bf16 --kv-unified --jinja --spec-type ngram-mod --spec-ngram-mod-n-match 24 --spec-ngram-mod-n-min 48 --spec-ngram-mod-n-max 64 --chat-template-kwargs '{"enable_thinking": true, "preserve_thinking": false, "reasoning_effort":"medium"}' --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.8-27B-GGUF/snapshots/f1bfb127c64f7072bdd2cad55f258b9c8b2910fe/mmproj-F16.gguf
```

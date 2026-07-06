# llama-cpp Configs


## Qwen 3.6 Q4_XL with 128k context window using Q5_1


```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/Qwen3.6-27B-UD-Q4_K_XL.gguf --port 35019 -c 128000 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads -1 --jinja --cache-type-k q5_1 --cache-type-v q5_1 --spec-default --chat-template-kwargs {"enable_thinking": true} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/mmproj-F16.gguf
```

### Split-mode - tensor (simple run)

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/Qwen3.6-27B-UD-Q4_K_XL.gguf --port 8888 -c 128000 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads 16 --split-mode tensor --fit off --jinja --cache-type-k bf16 --cache-type-v bf16 --spec-default --chat-template-kwargs '{"enable_thinking": true}' --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-27B-GGUF/snapshots/82d411acf4a06cfb8d9b073a5211bf410bfc29bf/mmproj-F16.gguf
```


## Qwen 3.6 35B-A3B Q4_XL with 128k context window

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-35B-A3B-GGUF/snapshots/a483e9e6cbd595906af30beda3187c2663a1118c/Qwen3.6-35B-A3B-UD-Q4_K_XL.gguf --port 48713 -c 262144 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads -1 --jinja --spec-default --chat-template-kwargs {"enable_thinking": false} --mmproj /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-35B-A3B-GGUF/snapshots/a483e9e6cbd595906af30beda3187c2663a1118c/mmproj-F16.gguf
```

### Split-mode - tensor (simple run)

```bash
llama-server -m /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-35B-A3B-GGUF/snapshots/a483e9e6cbd595906af30beda3187c2663a1118c/Qwen3.6-35B-A3B-UD-Q4_K_XL.gguf  --port 8888 -c 128000 --parallel 1 --flash-attn on --no-context-shift -ngl -1 --threads 16 --split-mode tensor --fit off --jinja --cache-type-k bf16 --cache-type-v bf16 --spec-default --chat-template-kwargs '{"enable_thinking": true}' --mmproj  /home/gitaroktato/.cache/huggingface/hub/models--unsloth--Qwen3.6-35B-A3B-GGUF/snapshots/a483e9e6cbd595906af30beda3187c2663a1118c/mmproj-F16.gguf
```

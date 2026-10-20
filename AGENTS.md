# AGENTS.md

Personal lab repo for running LLMs locally on custom hardware.
Repo: github.com/gitaroktato/local-llm

## Instructions

Any fundamental concept learnt from this project should be saved inside the root `AGENTS.md` file.

- Plan work ends with updating/creating the GitHub issue — do NOT execute the plan's steps inline; the issue is the deliverable, execution happens later from it.

## Performance Benchmarking

Usually the LLM conversations are itself served by the local llama-server, so per-turn timings are free to read — no external benchmark harness needed. The Studio build has no `llama-bench` binary; in-session measurement is the tool.

- Server logs: `~/.unsloth/studio/logs/llama-server/*.log`, one file per server session (`llama-<epoch>-port-<port>-try0.log`). Newest file = current live server.
- Live process + actual launch args: `ps -eo pid,etime,args | grep llama-server`. Cross-check against the newest log — trust the args in the log over the Studio UI selection.
- Per-turn speed: `print_timing` lines in the active log — `eval time = X ms / N tokens (T tok/s)`, plus `draft acceptance` and `mean len` for spec decoding.
- Spec config that actually ran: `spec common_specu: adding speculative implementation '<name>'` + its params, and `spec common_specu: statistics` lines. Acceptance × mean draft length is what determines effective tok/s, not raw engine speed.
- Build identification: each log starts with a build banner (build number + commit hash); the model snapshot hash is in the load section. Use both to attribute a perf change to binary vs config vs model.
- Gotcha: Studio spec "auto" resolves to NO implementation unless a drafter is available, and an unavailable drafter (e.g. missing `dflash-*.gguf` sidecar) silently falls back to `--spec-default` → ngram-mod. Always set the spec type explicitly and verify the log line before measuring.
- Flag choices for any build: `~/.unsloth/llama.cpp/llama-server --help` (e.g. `--spec-type` lists all supported implementations). Saved launch commands live in `llama-cpp-configs.md`.
- Prometheus endpoint `/metrics` (Studio launches with `--metrics`; port from `ps`): auth via `Authorization: Bearer $(cat ~/.unsloth/studio/auth/llama_api_key_*)`. Counters are cumulative since server start — measure as deltas between two scrapes. Useful extras over the logs: per-position draft acceptance (`spec_decode_num_accepted_tokens_per_pos_total{position=N}`) shows whether a deeper `--spec-draft-n-max` would pay off, and `prompt_tokens_cached_total` vs `prompt_tokens_total` gives KV cache reuse ratio.

## Privileged Commands (sudo)

The owner executes ALL `sudo` commands personally. The agent must NEVER run sudo itself (no passwordless probing either). Instead: prepare everything else locally, then hand the exact sudo commands to the owner as a copy-paste block and wait for confirmation before continuing.


## GitHub Issues

Use `gh` CLI. Check open issues before creating to avoid duplicates:
`gh issue list --state open`

- Create an issue for new work, plans, or bugs:
  `gh issue create --title "GPU power cap plan" --body "$(cat plan.md)" --label enhancement`
- Titles: short outcome phrase, no trailing period (e.g. "GPU power cap plan").
- Labels (apply when it fits; plain to-dos may stay unlabeled):
  bug → `bug`, feature → `enhancement`, docs → `documentation`, lessons/postmortems → `lessons`.
  Add later: `gh issue edit 21 --add-label lessons`
- Plan/investigation issues: Background → Findings → Decisions → Execution Steps → Deliverables → Revert.
- Bug issues: what happened, environment (OS/driver/CUDA/model), repro steps, expected vs actual.
- Read an issue: `gh issue view 21`
- Note progress in a comment: `gh issue comment 21 --body "Applied 165W cap, results pending"`
- Reference the issue in commits as `(#N)`. Do NOT use `Closes`/`Fixes` keywords — they auto-close the issue on merge.
- NEVER close an issue yourself (no `gh issue close`, no closing keywords anywhere). Work is delivered via a merge request: commit to a feature branch, then open `gh pr create --base main --title "Short outcome phrase" --body "Work for #N"` and stop. The owner reviews and merges the PR; only after that may the issue be closed, and by the owner — not you.
- Rejections: label `wontfix`, `duplicate`, or `invalid` and comment why; closing is left to the owner.
- Plan changed mid-work? Update the body: `gh issue edit 21 --body "$(cat updated-plan.md)"`

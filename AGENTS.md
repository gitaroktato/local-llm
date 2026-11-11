# AGENTS.md

Personal lab repo for running LLMs locally on custom hardware.
Repo: github.com/gitaroktato/local-llm

## Instructions

Any fundamental concept learnt from this project should be saved inside the root `AGENTS.md` file.

- Plan work ends with updating/creating the GitHub issue — do NOT execute the plan's steps inline; the issue is the deliverable, execution happens later from it.

## Session Logs

`scripts/export-sessions.sh` exports all opencode sessions of this project to `.sessions/<sessionID>.json` (gitignored). Re-runs skip up-to-date sessions and only re-export ones changed since their file's mtime; `--force` re-exports everything.

## Privileged Commands (sudo)

The owner executes ALL `sudo` commands personally. The agent must NEVER run sudo itself (no passwordless probing either). Instead: prepare everything else locally, then hand the exact sudo commands to the owner as a copy-paste block and wait for confirmation before continuing.

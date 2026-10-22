# AGENTS.md

Personal lab repo for running LLMs locally on custom hardware.
Repo: github.com/gitaroktato/local-llm

## Instructions

Any fundamental concept learnt from this project should be saved inside the root `AGENTS.md` file.

- Plan work ends with updating/creating the GitHub issue — do NOT execute the plan's steps inline; the issue is the deliverable, execution happens later from it.

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

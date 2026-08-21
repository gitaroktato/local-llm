# AGENTS.md

Personal lab repo for running LLMs locally on custom hardware.
Repo: github.com/gitaroktato/local-llm

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
- Reference the issue in commits: `Closes #N` / `Fixes #N`.
- Close with a one-line summary when done: `gh issue close 21 -c "Capped at 150W, ~3% perf loss"`
- Rejections: label `wontfix`, `duplicate`, or `invalid`, comment why, then close.
- Plan changed mid-work? Update the body: `gh issue edit 21 --body "$(cat updated-plan.md)"`

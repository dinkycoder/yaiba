# CLAUDE.md — Yaiba

Yaiba is risk tooling for Lightning edge nodes running Taproot Assets. It plugs into tapd as a Portfolio
Pilot and guards against the free American call option, with counterparty screening as a second
feature. The full plan, phase gates and dates are in [README.md](README.md). Read it before any task.

## How to work with the founder (Dinky)
- Teach as you build. Explain every new term and concept in plain English, in the style of an MSAIE
  lesson: what it is, why it matters for Yaiba, a worked example where useful, and a short
  check-your-understanding question at the end of a lesson.
- Keep answers executive-summary length. Use cause-and-effect sentences.
- Write "Yaiba" with a capital Y in prose. The repo path stays `dinkycoder/yaiba`.
- When asked to stop before the next lesson, stop and wait.

## Decisions already made
- Yaiba has its own repo. It does not use the SENT repo (`dinkycoder/sent`).
- Build first, test with operators after. The kill threshold in Phase 4 decides whether to pivot.
- Bitcoin and Lightning only. Taproot Assets first; RGB through an adapter once its Lightning support ships.
- Lead with free-option risk; compliance screening is second. Screening defaults to advise mode
  (accept, record a signed REVIEW_REQUIRED decision, alert the operator). Enforce mode is opt-in.
- A lab runs about one day a week alongside the phase plan (README "Lab practice"). Setting up
  `docs/lab/` (bet list plus five agent definitions) is a Phase 1 task.

## Open founder decisions
- Confirm or change the kill-threshold numbers, then Phase 0's `docs/prd.md` can be drafted.

## Git
- Keep the local clone and GitHub in sync: commit, then push, after every change.
- Verify tapd and Taproot Assets API names against the pinned source before using them; never invent them.

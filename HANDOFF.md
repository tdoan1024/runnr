# runnr — current handoff

Updated: 2026-11-13 (America/Los_Angeles)

## Where we are
- Discovery complete; technical design drafted, awaiting user review.
- Repository cloned locally. Only documentation exists; no app, dependencies, accounts, deployment, or AI integration created.
- Repository: https://github.com/tdoan1024/runnr
- Local: `C:/Users/Tai Doan/projects/tdoan1024/runnr`

## Agreed direction
- Separate mobile-friendly web app on Vercel, linked from the portfolio.
- Beginner running coach: conversational welcome, relevant follow-up questions, sign-in, saved conversations.
- Detailed video reasoning presented as 2–3 useful observations with timestamps, practical adjustments, and uncertainty; follow-up chat provides detail.
- Videos at most 15 seconds. Report template can evolve later.
- $0 initial budget; usage limits and a saved demo for portfolio visitors. User will record demo material later.
- Injury diagnosis is outside the first release; no sensitive injury uploads to unpaid AI services.

## Read next
- `docs/superpowers/specs/2026-11-13-runnr-design.md`: proposed architecture, constraints, milestones, and acceptance criteria.
- `AGENTS.md`: continuity rules for future sessions.

## Next session
1. Read this file and the design; inspect git status before editing.
2. Resolve the design's free-provider/personal-media constraint with the user and obtain design approval.
3. Write the implementation plan, then begin the small video feasibility experiment before building the full app.

## Open items / blockers
- Technology choices are proposals, not user-approved architecture.
- Free Gemini services cannot be treated as a general private health/video backend. Public demo uses reviewed nonsensitive examples; unrestricted personal uploads remain gated.
- Need provider availability/quota verification and a suitable demo clip before evaluating video accuracy.
- No credentials supplied. Never store credentials or private media in this repository.
- Builder review completed; the consequential open decision is restricted free showcase versus investigating private/local inference for personal inputs.
- Documentation-only validation: `git diff --check` passed. Check Git history/remote status for latest synchronization; no application tests exist yet.

## Updating this file
Replace stale status and next steps at the end of each work session or milestone. Keep this a compact current snapshot; put detailed reasoning in design documents and implementation history in Git.

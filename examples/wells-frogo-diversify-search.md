# Example — wells-frogo "search bar in the Diversify flow"

The **first real end-to-end Wave Build run** (2026-08-30), and the source of
the v0.2 → v0.3 changes. Run autonomously (`mode: autobuild`) by one Claude
Code session acting as Coordinator, spawning subagents for every other role.
Artifacts live in the `wells-frogo` repo under `.wavebuild/` (`phase-spec.md`,
`wave-plan.md`, per-wave `report.md` / `review.md` / `verify.md` /
`tests.log`, `progress.md`).

## The brief

> "I want a search bar in the diversify feature so I can look up a stock
> that I want to invest in."

The Diversify flow (`DiversifyFlow.tsx`) was sector → pick-from-auto-candidates
→ confirm. The user wanted to search any ticker and take it through the same
confirm/preview/apply path. One interview round (4 questions) settled: any
sector, search lives in the candidates step, show a score, buy-now (no
watchlist).

## The spec — 4 phases, total weight 8

| Phase | | wt | gate |
|---|---|---|---|
| **P0** | Contract: make `DiversifyPreviewRequest.sector` optional; extract the one shared `_resolve_diversify_target` helper both endpoints call — **behaviour-preserving, sector-given branch only** | 1 | full → *should have been* standard (finding 3) |
| **P1** | Backend: the `sector`-absent branch — resolve real sector server-side, size with no score, keep every guard | 3 | full |
| **P2** | Frontend: the search box in the candidates step, wired to the existing confirm path | 3 | standard |
| **P3** | Polish + docs | 1 | light |

## The partition — this is the point of Phase 0

```
Wave 1:  P0 alone
Wave 2:  P1 (backend) ∥ P2 (frontend)   ← real parallelism
Wave 3:  P3 alone (docs/polish)
```

P0 landed the contract, so P1 and P2 built **concurrently with zero file
collision and zero merge conflict** — P1 touched `main.py` + backend tests,
P2 touched `DiversifyFlow.tsx` + `api.ts` + frontend tests. Exactly the
shape §ACT 2 aims for.

## What the gates caught

- **W1 full gate**: Reviewer verified the extraction byte-for-byte;
  Verifier ran the suite on a clean checkout + exercised the trade path.
  Net value of the Verifier over the test gate: near zero — P0 changed no
  behaviour. → **finding 3**.
- **W2 full gate**: the fresh Reviewer found a **major** bug the Verifier's
  happy path missed — a thrown `/screener/score` call blocked the buy
  (should be display-only). The P2 builder had "deferred it to Phase 3" in
  a build note, but Phase 3's spec had no such item → an *untracked*
  deferral. → **finding 5**. One fix round, then a scoped re-review (not a
  second full Verifier pass).
- **W3 light gate**: Coordinator eyeballed the docs/copy/one-test diff.
  Enough.

Final: backend 419 → **433** tests, frontend 263 → **272**, all green,
merged to the working branch.

## Findings → v0.3

| # | What broke | v0.3 change |
|---|---|---|
| 1 | `Agent` tool worktree isolation unavailable (no VCS hooks) | `ADAPTERS.md`: manual `git worktree add` + symlink `.venv`/`node_modules` per worktree |
| 2 | `wave-N` (integration) vs `wave-N/phase-K` — can't both be git refs | integration branch is `wave-N-int` everywhere |
| 3 | §Phase 0's blanket `gate: full` wastes a Verifier on a pure refactor | `gate: full` only when Phase 0 lands behaviour; `standard` for an extraction/stub |
| 4 | Parallel builders both died on the session rate limit | §Speed: wave width is a *budget* lever, not only speed; resume, don't restart |
| 5 | Builder deferred a bug in a build note with nowhere tracking it | §Economy + Builder role: a deferral must be a `ponytail:` or a spec item |
| 6 | `.wavebuild/` already held an abandoned prior run | §Resuming: archive it to `.wavebuild/archive/<feature>/` first |
| 7 | Verifier's "clean checkout" had no DB / runtime data | Verifier role: seed from fixtures, never the user's live data |

## Takeaway

The core thesis held on a real repo: **Phase 0 bought genuine parallelism,
and context-isolated review caught a real bug self-review would have
missed.** Everything in the v0.3 table is mechanical friction around that
core, not a flaw in it.

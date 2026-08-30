# Wave Build — project map

Wave Build is a **platform-neutral protocol for building a multi-phase software
feature**: interview a messy brief into a phase spec, partition the phases
into waves that build in parallel where safe, and gate each wave behind an
independent review + verification. It ships as a Claude Code skill and as
plain markdown any agent (Codex, Cursor, Aider) or human can follow.

This project is **just documentation** — no code, no build, no test suite.
The deliverable is the method itself.

## Origin

Extracted 2026-08-30 from the `wells-frogo` repo (sibling directory
`../wells-frogo`), where it was first written inline as `.wavebuild/` while
planning that repo's "Reallocate" feature, then pulled out to stand alone.
Briefly renamed to "Beehive" (a throwaway standalone repo, `github.com/
hanifmohammed00/beehive`, now abandoned), then renamed back to **Wave Build**
and re-homed in a fresh `wave-build` repo. `wells-frogo` keeps only its own
per-project state under `.wavebuild/` and a pointer in its `CLAUDE.md`.

## Files — and which is authoritative

| File | Role | Authority |
|---|---|---|
| `PROTOCOL.md` | The method: three acts, Phase 0, wave partitioning, gates, §Speed, §Economy, §Progress, invariants | **Source of truth.** Everything else defers to it. |
| `ROLES.md` | Prompt templates: Interviewer, Coordinator, Builder, Reviewer, Verifier | Must stay consistent with `PROTOCOL.md`; it operationalises it |
| `ADAPTERS.md` | How to instantiate the roles per platform (Claude Code, Codex CLI/Cloud, Cursor/Aider, manual) | Platform mechanics only — no method decisions here |
| `SKILL.md` | Claude Code skill entry (`/wave-build`), `name: wave-build` | Thin dispatch wrapper; points at `PROTOCOL.md` |
| `README.md` | Human-facing: what it is, install, bootstrap | Keep in sync with `PROTOCOL.md` at a summary level |
| `templates/` | Starting `config.yml` / `brief.md` / `phase-spec.md` for a target repo's `.wavebuild/` | `config.yml` lists exactly the keys `PROTOCOL.md` §config.yml defines; `phase-spec.md` the fields §Input contract defines |
| `examples/` | Worked examples — `wells-frogo-diversify-search.md` (the first real v0.3 run, norm-shaped), `wells-frogo-reallocate.md` (pre-protocol retrospective) | Illustrative, not normative |
| `LICENSE` | MIT | — |
| `CLAUDE.md` | This file | Orientation only |

## Core invariants — do not break these when editing

1. **Roles are defined by context isolation, not by tool.** Reviewer and
   Verifier must be fresh instances that never saw a Builder's reasoning.
2. **Speed comes from widening the dependency graph** (Phase 0 contracts,
   merging hot-file-colliding phases), not from adding builders to a chain.
3. **Every gate protects a guarantee.** Don't add a speed optimisation that
   skips a test run, builds on unverified code, or reviews a combined diff
   instead of a scoped one. See `PROTOCOL.md` §Speed "what does not help"
   and the chat rationale for why skip-scope caching / speculative builds /
   persistent verify envs were rejected from the core.
4. **All cross-role handoff is files under `.wavebuild/`** — so any platform
   or session can resume. No mechanism that only works in-memory or
   in one tool.
5. **Economy applies to the protocol's own prose too.** If an edit makes a
   section longer without adding a rule, it's probably wrong.

## Editing this project

- Change `PROTOCOL.md` first, then propagate to `ROLES.md` / `SKILL.md` /
  `README.md` / `templates/`. Grep for the term you changed.
- Keep the version line at the top of `PROTOCOL.md` current (`v0.3` now).
  Bump minor for a new rule or field; note what changed in the `<!-- -->`
  comment under the version line.
- Step numbers in `PROTOCOL.md` §ACT 3 are referenced from `ROLES.md` and
  §Speed — if you renumber, fix the references.
- There's nothing to run. To actually test a change, use `/wave-build` on a
  real feature in a real repo and watch where it goes wrong — the v0.3
  changes all came from doing exactly that (`examples/wells-frogo-diversify-search.md`).

## Install / use

`README.md` has it. Short version: `ln -s "$(pwd)" ~/.claude/skills/wave-build`
then `/wave-build` in any project. Private repo:
`github.com/hanifmohammed00/wave-build` (MIT-licensed). `v0.3` current.

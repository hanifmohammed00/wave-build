---
name: wave-build
description: >
  Run the Wave Build Protocol — interview a feature brief into a phase spec,
  partition it into parallel-safe waves, and drive each wave through
  build → integrate → test → independent review → verify, autonomously,
  with a weight-based completion estimate after every wave. Use when the
  user wants to build a multi-phase feature with quality gates, says
  "wave build", "run wave build", "waved build", or points at
  .wavebuild/phase-spec.md. Also use to resume an in-progress run.
argument-hint: "[intake | plan | build | resume]"
---

# Wave Build

Thin wrapper. The method is `PROTOCOL.md` (in this skill directory) — read
it in full, then act as the **Coordinator** role from `ROLES.md`. Platform
mechanics for spawning Builders / Reviewers / Verifiers are in
`ADAPTERS.md`.

Per-project state lives in the target repo's `.wavebuild/` directory:
`config.yml`, `brief.md`, `phase-spec.md`, `wave-plan.md`, `progress.md`,
`wave-N/`. If that directory has no `config.yml`, start at intake and
create it — starting points for `config.yml`, `brief.md`, and
`phase-spec.md` are in this skill's `templates/`.

## Dispatch

- **no arg / `resume`** — read the target repo's `.wavebuild/` state
  (`PROTOCOL.md` §Resuming): newest `wave-*/report.md`, `status`,
  `progress.md`. Continue from there. No `phase-spec.md` → start at intake.
- **`intake`** — run `ROLES.md` §Interviewer against `.wavebuild/brief.md`
  (create it with the user first if missing). If `.wavebuild/` still holds a
  completed/abandoned prior run, archive it to `.wavebuild/archive/<feature>/`
  first. Produces `phase-spec.md` (with a contracts-first Phase 0),
  `config.yml`, and `digest.md`. Ends by setting `mode:` from the "read the
  plan or just build?" question (default `autobuild`).
- **`plan`** — `PROTOCOL.md` §ACT 2: partition `config.yml`'s `spec` into
  waves, write `.wavebuild/wave-plan.md`.
- **`build`** — `PROTOCOL.md` §ACT 3 wave by wave. `mode: autobuild`
  (default) runs every phase and wave to completion without stopping for
  permission, updating `.wavebuild/progress.md` after each wave, stopping
  only on an unrecoverable block. `mode: review` also stops for approval
  at each gate.

## Non-negotiable

- Obey `PROTOCOL.md` §Economy and §Invariant rules — Reviewer and Verifier
  are always fresh subagents, never a Builder; Builders get only their
  phase's scope; test output is pasted raw; two phases sharing a `hot_file`
  never share a wave.
- The Coordinator does not write feature code — only merges, test runs,
  reports, progress, and doc edits.
- Never invoke `config.yml` `heavy_review` — name it in the wave report for
  the user to run.

# Wave Build

A platform-neutral protocol for building a multi-phase feature: interview a
messy brief into a phase spec, partition it into **waves** that build in
parallel where safe, and gate every wave behind an independent review and a
real end-to-end verification — autonomously, with a completion estimate
after each wave.

MIT-licensed. Vendor `PROTOCOL.md`, `ROLES.md`, `ADAPTERS.md` into any repo.

Works with Claude Code, Codex (CLI or Cloud), Cursor, Aider, or a human
coordinating several agent sessions. All state is plain files in the target
repo's `.wavebuild/` directory, so any agent on any platform can resume a run.

## Contents

| File | What it is |
|---|---|
| `SKILL.md` | Claude Code skill entry (`/wave-build`) |
| `PROTOCOL.md` | The method — three acts, contracts-first Phase 0, wave partitioning, risk-proportional + pipelined gates, economy rules, progress formula, §Speed. Vendor this unchanged. |
| `ROLES.md` | Prompt templates for Interviewer, Coordinator, Builder, Reviewer, Verifier |
| `ADAPTERS.md` | How to instantiate the roles on each platform |
| `templates/` | Starting `config.yml`, `brief.md`, and `phase-spec.md` for a target repo's `.wavebuild/` |
| `examples/` | Worked examples |

## Install as a Claude Code skill

Global (available in every project):

```bash
ln -s "$(pwd)" ~/.claude/skills/wave-build
```

Or per-project:

```bash
ln -s /path/to/wave-build /path/to/your-repo/.claude/skills/wave-build
```

Then `/wave-build` in that project. A copy works too — re-copy on update.

## Use without a skill (Codex etc.)

Point the repo's `AGENTS.md` at the protocol:

```
Multi-phase features follow wave-build: PROTOCOL.md — interview to a phase
spec, build in waves with independent review + verify gates.
```

Vendor `PROTOCOL.md`, `ROLES.md`, `ADAPTERS.md` into the repo (e.g. under
`.wavebuild/`) so the agent can read them.

## Bootstrapping a run

1. `mkdir .wavebuild` in the target repo; copy `templates/config.yml` to
   `.wavebuild/config.yml` and fill it (hot files, test commands, context).
2. Copy `templates/brief.md` to `.wavebuild/brief.md` and fill it in —
   everything you want, however messy.
3. Run `/wave-build intake` (or hand `ROLES.md` §Interviewer to any agent, or
   write `.wavebuild/phase-spec.md` yourself from `templates/phase-spec.md`).
   It interviews you, then writes `.wavebuild/phase-spec.md` (with a
   contracts-first Phase 0 so the rest fans out in parallel),
   `.wavebuild/digest.md` (shared context every later agent reads instead of
   re-exploring), and asks whether to just build or let you review first.
4. It partitions into waves and builds — Builders in a wave run
   concurrently, gates are as deep as each phase's risk warrants, and the
   next wave's build overlaps the current wave's review. Reports after each
   wave; writes a completion percent to `.wavebuild/progress.md`.

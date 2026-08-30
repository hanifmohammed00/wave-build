# Wave Build

A method for getting an AI agent to build a big, multi-part feature without
the usual mess. It's just markdown. No install beyond a symlink, nothing to
run.

## Why

Give an agent a large feature and it usually does one of two things: dumps
one giant diff on you, or builds one piece at a time even when the pieces
don't depend on each other. And whoever reviews the result already knows
what the author was thinking, so the review goes soft.

## What it does

- **Splits the work so it can run in parallel.** It pulls the shared pieces
  (types, schemas, function signatures) into a "Phase 0" and builds those
  first. Everything else depends only on Phase 0, so the rest can be built at
  the same time instead of one after another.
- **Reviews each batch with a fresh agent.** The reviewer and tester never
  saw the code being written. They get the diff and the spec, nothing else.
  Risky changes get a full review and a real end-to-end test; a one-line doc
  change gets a quick look.
- **Runs on its own and reports back.** It goes batch to batch without asking
  permission, with a status update and a how-far-along number after each
  one. It stops only if tests fail, a check fails, or two pieces turn out to
  disagree.
- **Keeps everything in files.** All the state lives in a `.wavebuild/`
  folder in your repo, so any agent or session can pick up where another
  left off.

## How it works

1. **Intake.** You write a rough `brief.md`, whatever you want, however
   messy. An agent reads your repo, asks you questions in batches, and turns
   it into a phase-by-phase spec.
2. **Plan.** The phases get grouped into waves. Independent phases share a
   wave and run together; phases that would fight over the same file get
   their own.
3. **Build.** For each wave: build the phases in parallel, merge them, run
   the full test suite, review and verify, merge to your branch. The last
   wave is always cleanup and docs.

You end up with a merged feature and a trail: the spec, every test run,
every review.

## When to use it

Good for a feature with four or more phases, especially when some are
independent (a backend change and a frontend change that only meet at an API
contract). Skip it for small changes. The setup isn't worth it.

## Install

```bash
ln -s "$(pwd)" ~/.claude/skills/wave-build
```

Then run `/wave-build` in any repo. For non-Claude agents, point the repo's
`AGENTS.md` at `PROTOCOL.md`.

## Start a run

1. Make a `.wavebuild/` folder in your repo. Copy `templates/config.yml` and
   `templates/brief.md` into it and fill them in.
2. Run `/wave-build intake`.
3. It interviews you, writes the plan, and builds, asking first whether you
   want to check the plan or just go.

## Status

v0.3, hardened after its first full run on a real feature. The changes are
listed at the top of `PROTOCOL.md`.

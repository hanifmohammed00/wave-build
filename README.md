# Wave Build

A protocol for building a large feature with AI agents without the usual
failure modes. It's plain markdown, no dependency, nothing to run. You point
an agent at it and it drives the build.

## The problem it solves

Hand an agent a multi-phase feature and you tend to get one of two bad
outcomes. It builds everything in one pass and hands you a 2,000-line diff
where you can't tell which part is load-bearing. Or it builds phase by phase
in a strict chain, each phase idling until the previous one lands, so a
feature with three independent pieces still takes three times longer than it
should.

The review is usually weak too. It's either the same agent that wrote the
code or a fresh one that got handed the author's notes, and either way it
nods along.

## What it gives you

### Parallel work that doesn't collide

Before any feature code gets written, an interview pass pulls every shared
type, schema, and function signature into a single Phase 0. Everything else
then depends on Phase 0 and nothing else, so the rest of the work fans out
into one wave that builds at the same time. On the first real run, the
backend and frontend halves of a feature were built concurrently with no
merge conflicts.

### A review that's genuinely independent

The reviewer and verifier are fresh agents. They never watched the builder
work and they don't get its transcript, just the diff and the spec. On that
same run this caught a bug the builder had quietly written off in a note: a
failed API call that blocked a purchase, in code that had already passed its
own tests.

The checks also scale with the stakes. A one-line docs phase gets a glance;
the 400-line phase that moves money gets the fresh reviewer, a fresh
verifier, a clean-checkout boot, and someone exercising the real feature.

### It runs on its own but stops when it should

The default mode goes wave to wave with no permission stops. After each wave
it writes a report and a completion percentage instead of asking what to do
next. It stops for you in three cases: tests still failing after the
builder's own retries, a failed end-to-end verification, or a merge conflict
that means the spec was wrong.

Every handoff is a plain file under `.wavebuild/` in your repo, so a run
isn't tied to one tool or one session. Another agent picks it up by reading
the directory, whether that's Claude Code, Codex, Cursor, or a person
running a few terminal windows.

## How it works

Three acts.

**Intake.** You write `brief.md`: everything you want, in whatever messy form
it comes out. That's the only writing you have to do. An interviewer agent
reads the relevant parts of the repo, drafts a phase breakdown, and asks you
questions in batches until it can pin down every phase's scope with nothing
left to guess. Then it lifts the shared contracts into Phase 0 and writes
the spec.

**Plan.** A coordinator turns the phase list into waves. The rule that
matters: two phases that both edit the same oversized file never share a
wave. Phase 0 runs first, alone. Everything that needs only Phase 0 goes in
the next wave, together. If the plan still comes out as a long chain, Phase 0
missed a shared piece.

**Build.** For each wave, the coordinator starts every builder in it at once,
each on its own branch. It merges them, runs the full test suite, then
reviews and verifies at the depth the wave's riskiest phase calls for, and
merges to your working branch. The next wave's builders start while the last
wave's review is still running in the background. Reviews don't edit code, so
a finding comes back as a fix on the branch that already merged.

The final wave is always docs and cleanup, by itself. When it finishes you
have a merged feature and a full trail: the spec, each wave's test output,
every review, every verification.

## When it's worth it

Reach for it on a feature with four or more phases, particularly when some of
them are independent (a backend change and a frontend change that only meet
at an API contract). It also helps any time you'd rather find a foundation
bug at wave 2 than after building three more waves on top of it.

Don't bother for a one-file fix or a change with no independent parts. There
is real setup cost here, and on something small it will outweigh the work.

## Contents

| File | What it is |
|---|---|
| `PROTOCOL.md` | The method. Source of truth. Vendor this unchanged. |
| `ROLES.md` | Prompt templates for Interviewer, Coordinator, Builder, Reviewer, Verifier |
| `ADAPTERS.md` | How to run the roles on each platform |
| `SKILL.md` | Claude Code skill entry (`/wave-build`) |
| `templates/` | Starting `config.yml`, `brief.md`, `phase-spec.md` |

## Install

As a Claude Code skill, available in every project:

```bash
ln -s "$(pwd)" ~/.claude/skills/wave-build
```

Then `/wave-build` in any repo. For other agents, point the repo's
`AGENTS.md` at `PROTOCOL.md` and copy `PROTOCOL.md`, `ROLES.md`, and
`ADAPTERS.md` into it.

## Bootstrapping a run

1. `mkdir .wavebuild` in the target repo. Copy `templates/config.yml` to
   `.wavebuild/config.yml` and fill in the hot files, test commands, and
   context.
2. Copy `templates/brief.md` to `.wavebuild/brief.md` and write down what
   you want.
3. Run `/wave-build intake`. It interviews you, writes the phase spec with a
   contracts-first Phase 0, and asks whether to just build or let you review
   the plan first.
4. It partitions into waves and builds, reporting a completion percentage
   after each one.

## Status

v0.3. Short history: drafted while planning a feature, then run start to
finish on a real one. That run broke it in seven places, all since fixed and
listed in the comment at the top of `PROTOCOL.md`.

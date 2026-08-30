# Wave Build — platform adapters

The protocol needs three things a platform must provide its own way:
**(a)** a Builder scoped to one phase, **(b)** a Reviewer and Verifier that
are fresh instances with no exposure to the Builder, **(c)** optionally,
parallel Builders. Everything else is plain git + shell + files.

---

## Claude Code

- **Coordinator**: the main session, or `/wave-build` (skill wrapper reads
  `config.yml` and runs the Coordinator role).
- **Builder**: `Agent` tool, `subagent_type: general-purpose`, one call per
  phase. Parallel builders in one wave → `isolation: "worktree"` +
  `run_in_background: true`, then collect. `model: {models.strong}`.
  - If `isolation: "worktree"` isn't available (it needs VCS hooks the
    environment may not have), fall back to **manual worktrees**: the
    Coordinator runs `git worktree add -b wave-N/phase-K <dir> <base>` per
    phase and passes each Builder its `<dir>`. Symlink the gitignored build
    deps into each worktree first (`ln -s <repo>/backend/.venv <dir>/backend/.venv`,
    same for `node_modules`) or tests won't run. Background subagents are
    still fresh contexts, so isolation holds. Remove the worktrees
    (`git worktree remove --force`) and delete the merged branches after
    each wave.
  - **Resuming a dead Builder**: `SendMessage` to the same subagent — its
    context and its partial work on the branch are intact. A fresh `Agent`
    call starts cold and re-explores; only do that if the transcript is
    gone.
- **Reviewer / Verifier**: a new `Agent` call — a fresh subagent starts cold,
  which satisfies isolation. Never reuse a builder subagent for review.
  Verifier and light-gate Reviewer take `model: {models.cheap}`. Point them
  at `wave-N-int` (the integration branch) / its diff.
- **Concurrent tests**: run the `config.test` commands as parallel
  background `Bash` calls, then read both results.
- **Heavy review**: the human runs `config.heavy_review` (e.g.
  `/code-review ultra`); the Coordinator only names it in the wave report.

## Codex — CLI (`codex` / `codex exec`)

- **Coordinator**: the interactive `codex` session.
- **Builder**: `codex exec` per phase, sequential, prompt = `ROLES.md`
  §Builder with slots filled. Each writes its own branch.
- **Reviewer / Verifier**: a *separate* `codex exec` run pointed at the
  diff (`git diff <working-branch>...wave-N-int`) — new process, no shared
  context, so isolation holds. Prompt from `ROLES.md`.
- **Parallel**: not native; the human can launch multiple Codex Cloud tasks
  (below) for one wave's phases and merge the branches.

## Codex — Cloud tasks

- **Builder**: one cloud task per phase in the wave, prompt from `ROLES.md`
  §Builder — these run in parallel natively, each producing a branch/PR.
- **Integrate**: the human (or Coordinator task) merges the phase branches.
- **Reviewer / Verifier**: a separate cloud task per role, scoped to the
  merged branch's diff.
- No skill needed — point `AGENTS.md` at `.wavebuild/PROTOCOL.md`.

## Cursor / Aider / other single-session tools

- **Coordinator + Builder**: the session, one phase at a time.
- **Reviewer / Verifier**: a new chat (Cursor) or a fresh `aider` invocation
  (`aider --message "$(fill ROLES.md §Reviewer)"`) against the diff — new
  context = isolation.
- **Parallel**: none; run phases sequentially within a wave.

## Manual multi-agent (any mix of vendors)

- Human is the Coordinator. Open one agent window per phase for Builders,
  merge their branches, then open a *different* window (any vendor) for the
  Reviewer and another for the Verifier, pasting the role prompts.
- The `.wavebuild/` files are the shared state — every window reads and
  writes there.

---

## AGENTS.md / CLAUDE.md pointer

Add once, so any tool entering the repo finds the method:

```
Multi-phase features follow .wavebuild/PROTOCOL.md — interview to a
phase-spec, build in waves with independent review + verify gates.
```

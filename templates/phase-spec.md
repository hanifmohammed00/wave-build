# Phase spec — <feature name>

Produced by the Interviewer (Act 1), or hand-written. One numbered phase per
section; fields are verbatim per `PROTOCOL.md` §Input contract. Anything not
confirmed with the human is tagged `ASSUMED — correct me` inline, so it gets
fixed in review rather than in more Q&A.

Global rules: <conventions every phase must follow — package layout, commit
discipline, "recompute server-side, never trust client-echoed values", one
atomic commit per apply, etc. State them once here, not in every phase.>

---

## Phase 0 — Shared contracts

- **goal**: land every shared type / schema / migration / signature that two
  or more later phases need, so the rest `depends_on: [0]` instead of on
  each other.
- **deliverables**: shared request/response schemas, DB migration, helper
  signatures — the interface only, minimal or no behaviour.
- **touches**: <schema / types / migration files>
- **depends_on**: none
- **tests**: types compile / import cleanly; migration applies and reverts.
- **weight**: 2
- **gate**: full        # full if Phase 0 lands behaviour; standard if it's a
                        # behaviour-preserving extraction / signature-only stub

## Phase 1 — <name>

- **goal**: <one sentence>
- **deliverables**:
  - <what code and tests exist when this phase is done>
- **touches**: <file globs this phase may modify — the scope fence>
- **depends_on**: [0]
- **tests**: <the concrete checks that must pass for this phase>
- **weight**: 3         # 1 trivial … 8 major build
- **gate**: standard    # optional; omit to derive from weight (1–2 light, 3–4 standard, 5+ full)

## Phase 2 — <name>

- **goal**: …
- **deliverables**:
  - …
- **touches**: …
- **depends_on**: [0]
- **tests**: …
- **weight**: 3

## Phase N — Polish & docs

- **goal**: tidy the seams and record the feature.
- **deliverables**:
  - <relabel / tidy items; docs — README, handoff notes, API map updated>
- **touches**: <docs + polish files>
- **depends_on**: <the feature phases>
- **tests**: existing suite stays green; <one cross-cutting consistency check>
- **weight**: 1

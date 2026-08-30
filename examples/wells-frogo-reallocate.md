# Example — wells-frogo "Reallocate" feature

Planning-only retrospective, written against a **pre-protocol** draft (the
feature was built outside Wave Build). For a real end-to-end run see
`wells-frogo-diversify-search.md`. Kept for one lesson the other example
doesn't show: how `hot_files` cap parallelism.

Shape of it:

- **6 phases**, weights `P1:3 P2:3 P3:2 P4:2 P5:8 P6:1` (total 19).
- **hot_files**: `backend/app/main.py`, `backend/app/schemas.py`,
  `frontend/src/components/DiversifyFlow.tsx`.
- **Wave partition**: Wave 1 = P1 (backend) ∥ P2 (frontend) — no file
  overlap. Waves 2–3 = P3 then P4 (both hit all three hot files → can't
  share a wave). Wave 4 = P5. Wave 5 = P6 (docs, alone).
- Only Wave 1 parallelises; the rest is a dependency chain.

Takeaway: parallelism is bounded by hot files, not by how many phases have
their dependencies met. Three "ready" phases that all edit one 3,000-line
file still run one per wave.

The fix the protocol prescribes: a **Phase 0** landing the shared
`schemas.py` fields + `_credit_cash`/`_debit_cash` signatures so P3/P4/P5
depend on P0 only, and **merging** P3+P4 (serialized purely by the shared
hot files) into one phase for one Builder. That turns P0 → {P1, P2, P3+4}
→ P5 → P6 — ~6 serial units down to ~4, middle wave 3-wide. §Speed
levers 1–3.

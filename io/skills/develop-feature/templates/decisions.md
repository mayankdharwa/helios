# Decisions

<!--
Canonical entry shape — authoritative. Every procedure that appends a DECISIONS entry
(topic-lock, section-archive, surgical-reopen, cross-feature-lift) uses this shape verbatim.

## #N — <short title>

*Lifted: YYYY-MM-DD · Source: [<path>/<file> §X Review comment #M](<path>) or [build/code-review/<section>.md item "<one-line>"] or <as determined for repo-root originated entries>*

[*Superseded by #L — YYYY-MM-DD*]

[**Supersedes:** #M[, #K, …]]

**Decision:** <one sentence>

**Why:** <one paragraph including alternatives>

**Implications:** <optional>

Numbering:
- Per-feature `DECISIONS.md`: `#N` is per-feature monotonic. Read the file, take highest existing `#N`, use N+1.
- Repo-root `docs/DECISIONS.md`: independent counter, `#N` resets per file.

Supersession fields (both optional, bracketed in the shape above):
- `*Superseded by #L — YYYY-MM-DD*` — added to the older entry, directly below its `*Lifted: ... · Source: ...*` line. Italicised so it visually attaches to the meta block.
- `**Supersedes:** #M[, #K, …]` — added to the newer entry, directly above its `**Decision:**` line. Lists every entry being superseded.

Both are written by `procedures/supersede.md`. Supersession does not renumber. The older entry stays at its original `#M`, so existing backlinks remain valid; the marker above tells the reader where to look next. See `procedures/supersede.md` for the procedure and `reference/lift-criteria.md` for when supersession is the right move (vs. extending a prior decision in place).

The inline source (Review comment or code-review item) stays in place — never deleted or rewritten. DECISIONS.md is the executive summary with a backlink; the inline doc carries the full context.
-->

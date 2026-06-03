# Lift one decision

Append a single decision to `DECISIONS.md`. Called by `procedures/advance.md` during sweeps and by the immediate-lift path (which can fire any turn).

## Preconditions

- A feature dir is active.
- The candidate is one of:
  - A `Review comment #N` callout in `PLAN.md` (`## Exploration notes` or `## Approach`).
  - A `decision: <text>` resolution tag in `REVIEW.md` or `PLAN.md` `## Review notes`.
- The user has confirmed this specific decision should be lifted (per-item, not batched implicitly).

## Lift criteria

All three must hold. See `io:develop-feature` `reference/lift-criteria.md` for the canonical wording and edge cases.

1. **Hard to reverse** — reversing it would touch multiple unrelated files or phases.
2. **Surprising without context** — a future reader will ask "why did they do it this way?" without the rationale.
3. **Real trade-off** — there were genuine alternatives and one was picked for specific reasons.

If any fails, leave inline. Expect ~10–20% of callouts/tags to qualify.

## Steps

### 1. Confirm the lift with the user

Show the candidate (one-line summary of the callout/tag) and confirm all three criteria. Wait for an explicit yes for *this* item — even when called from a sweep that already got a batch yes, surface anything where the criteria check is borderline.

### 2. Create DECISIONS.md if missing

Path: `docs/<feature>/DECISIONS.md`. If it doesn't exist, create it from `io:develop-feature` `templates/decisions.md` verbatim. This file is shared shape with the full skill — graduate.md does not have to rewrite it.

### 3. Determine #N

Read existing `DECISIONS.md`, find the highest existing `## #M`, use `M+1`. Per-feature monotonic. Numbers never reuse, even if a prior decision is later reversed.

### 4. Append the entry

Canonical shape (from full skill's `templates/decisions.md`):

```markdown
## #N — <short title>

*Lifted: YYYY-MM-DD · Source: [PLAN.md §<section> Review comment #M](PLAN.md) or [REVIEW.md item "<one-line>"](REVIEW.md)*

**Decision:** <one sentence>

**Why:** <one paragraph including alternatives>

**Implications:** <optional>
```

`Source:` is a backlink to where the inline callout/tag lives. Mini's `Source:` paths point to `PLAN.md` or `REVIEW.md` directly (full skill points into `exploration/<topic>/*-EXPLORATION.md` or `build/code-review/<section>.md`). On graduation, these paths are rewritten by `procedures/graduate.md`.

### 5. Inline callout stays put

Never edit, move, or delete the source callout/tag — it carries the full context. The pointer is one-way: `DECISIONS.md → inline`.

### 6. Summarise

One line: which `#N` landed where, what it points to.

## Errors

- `DECISIONS.md` exists but has malformed numbering: surface, ask user before appending.
- Candidate callout's `Why:` field is empty: surface, ask user to fill it inline before lifting.

## Postconditions

- `DECISIONS.md` exists at `docs/<feature>/DECISIONS.md`.
- Exactly one new `## #N` entry has been appended with the canonical shape and a working `Source:` backlink.
- The inline callout/tag is unchanged.

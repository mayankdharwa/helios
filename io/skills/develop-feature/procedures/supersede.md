# Supersede a prior decision

Mark an older `DECISIONS.md` entry as superseded by a newer one. Adds a `*Superseded by #N — YYYY-MM-DD*` marker to the older entry and a `**Supersedes:** #M[, …]` line to the newer one. Numbers never reuse — the older entry stays at its original `#M` so existing backlinks keep working; the marker tells readers where the live decision now lives.

Distinct from in-place revision. Editing a prior `Decision:` / `Why:` / `Implications:` paragraph is **never** permitted (every DECISIONS entry is append-only). Supersession is the single carve-out: it touches only the marker line on the older entry and adds the `Supersedes:` line on the newer one. The original rationale stays untouched and remains readable.

## Preconditions

- Both entries (`#M` old, `#N` new) live in the same `DECISIONS.md` file. Cross-file supersession (per-feature → repo-root, or vice versa) is unusual — see Errors.
- The newer entry `#N` is already in the file. If the user is lifting a new decision *and* wants it to supersede an older one, run the lift path first (`procedures/topic-lock.md`, `procedures/section-archive.md`, `procedures/surgical-reopen.md`, or `procedures/cross-feature-lift.md` for repo-root-originated entries); invoke this procedure immediately after.
- The user has confirmed `#N` **replaces** `#M`, not merely refines or extends it.
- The older entry is not already marked superseded by a different entry — if it is, that's a chain (see Errors).

## Steps

### 1. Confirm the supersession (judgment)

Show one-line summaries of `#N` and `#M`. Apply the boundary:

- **Supersedes** if the older decision is no longer in force in any part. Reading only `#N`, a future reader gets the complete current answer.
- **Does NOT supersede** if the older decision still partly holds, or `#N` adds a constraint / refines scope / extends to a new case. In that case the right move is usually a fresh Review comment in the source doc plus an entry whose `Why:` paragraph references the prior `#M` — not a supersession marker.

Surface to user:

> Proposed supersession:
> `#N — <title>` replaces `#M — <title>`.
> Old decision is no longer in force? **<Y / N>**.
> Confirm / cancel?

If user cancels or the older still partly holds, stop. No edits.

A single newer entry may supersede multiple older entries — confirm each candidate `#K` separately.

Step 1's confirmation discharges DECISIONS.md's confirm-before-edit rule (`reference/doc-ownership.md`) for every write below; do not re-prompt before steps 2–3.

### 2. Add the `Superseded by` marker to `#M` (and any others confirmed in step 1)

For each older entry, locate its `*Lifted: ... · Source: ...*` italic meta line and insert directly **below** it:

```
*Superseded by #N — YYYY-MM-DD*
```

Italicised so it visually attaches to the meta block. Today's date is the supersession date, not `#N`'s lift date (they may differ).

Do **not** touch `Decision:`, `Why:`, `Implications:`, or `Source:` on the older entry. The original rationale stays intact and remains readable.

Older marker is written before the newer `Supersedes:` line so that a mid-procedure cancel leaves the more graceful partial state: `#M` points at the live `#N`, which already exists.

### 3. Add the `Supersedes:` line to `#N`

Read `DECISIONS.md`. Locate the `#N` block. Insert directly **above** the `**Decision:**` line:

```
**Supersedes:** #M[, #K, …]
```

List every entry being superseded by this one, comma-separated, in ascending numerical order. If `#N` already carries a `Supersedes:` line from a prior invocation, append the new numbers to that line rather than writing a second one.

### 4. Inline callouts and tags stay in place

Never edit, move, or delete the `Review comment` callout or `decision:` tag in the source doc that originated `#M`. The inline source still carries the original context. Readers discover the supersession via `DECISIONS.md`, then follow the original `Source:` link if they want the full history.

### 5. Cross-feature ripple check

If `#N` lives in a per-feature `DECISIONS.md` and the older `#M` had been promoted to repo-root via `procedures/cross-feature-lift.md`, the repo-root copy is now stale. Surface this and ask whether to:

- (a) supersede the repo-root entry as well (run this procedure again against repo-root, with `#N` re-lifted to repo-root first if it isn't already), or
- (b) leave repo-root unchanged because the cross-feature decision still holds in its broader scope (rare).

If the user picks (a), route to `procedures/cross-feature-lift.md` for the new entry, then re-enter this procedure pointing at the repo-root file. Do not write to repo-root from this procedure call.

### 6. Summarise

One line per superseded entry: `#N supersedes #M — marker added to #M, "Supersedes: #M" added to #N.` Note the cross-feature outcome from step 5 if it applied.

## Errors

- **Chain**: `#M` is already marked superseded by some `#L`. Surface the chain (`M → L → …`) and ask the user whether `#N` should supersede the head of the chain (typically yes — point `#N` at `#L` instead of `#M`) or whether the user wants `#N` to actively replace `#L` *and* re-supersede `#M` (mark both with `Superseded by #N`). Do not auto-resolve.
- **Cross-file**: the two entries are in different `DECISIONS.md` files (one per-feature, one repo-root). Surface and ask. Usually the right move is to supersede only within one file; see step 5's cross-feature ripple handling for the proper path.
- **Numbering drift**: `#M` or `#N` doesn't actually exist at the cited number (typo, hand-edit). Surface; ask the user to clarify before writing.
- **User cancels mid-procedure** (e.g., after step 2's marker lands on `#M` but before step 3 writes the `Supersedes:` line on `#N`): the half-written state is still readable — `#M` points at the live `#N`. Surface what landed and ask whether to roll back the marker on `#M` or proceed with step 3.

## Postconditions

- The newer entry `#N` carries a `**Supersedes:** #M[, …]` line directly above `**Decision:**`, listing every entry confirmed in step 1.
- Each superseded entry carries a `*Superseded by #N — YYYY-MM-DD*` italic line directly below its `*Lifted: ... · Source: ...*` line.
- The older entries' `Decision:` / `Why:` / `Implications:` / `Source:` are unchanged.
- Inline `Review comment` callouts / `decision:` tags in source docs are unchanged.
- Decision numbering in the file is unchanged — no entries renumbered, no entries deleted.
